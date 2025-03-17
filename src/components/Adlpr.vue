<script setup lang="ts">
import { onMounted, onBeforeUnmount } from "vue";
import * as THREE from "three";
import { Pane } from "tweakpane";

let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let letters: THREE.Mesh[] = [];
let frameId: number;
let mouse = new THREE.Vector2();

// Configuration object
const config = {
  rotationFactor: 2.0,
  pushStrength: 0.5,
  letterSpacing: 0.6,
  letterSize: 0.8,
  fontSize: 80,
  fontColor: "#ffffff",
  backgroundColor: "#000000",
  ambientLightIntensity: 0.5,
  pointLightIntensity: 1.0,
  pointLightPosition: { x: 5, y: 5, z: 5 },
};

const initGui = () => {
  const pane = new Pane();

  const animationFolder = pane.addFolder({ title: "Animation" });
  animationFolder.addBinding(config, "rotationFactor", {
    min: 0,
    max: 5,
    label: "Rotation Speed",
  });
  animationFolder.addBinding(config, "pushStrength", {
    min: 0,
    max: 2,
    label: "Push Strength",
  });

  const letterFolder = pane.addFolder({ title: "Letters" });
  letterFolder
    .addBinding(config, "letterSpacing", { min: 0.1, max: 2 })
    .on("change", updateLetterPositions);
  letterFolder
    .addBinding(config, "letterSize", { min: 0.1, max: 2 })
    .on("change", updateLetterSize);
  letterFolder
    .addBinding(config, "fontSize", { min: 20, max: 120 })
    .on("change", recreateLetters);
  letterFolder
    .addBinding(config, "fontColor", { view: "color" })
    .on("change", recreateLetters);

  const lightingFolder = pane.addFolder({ title: "Lighting" });
  lightingFolder
    .addBinding(config, "ambientLightIntensity", { min: 0, max: 2 })
    .on("change", updateLights);
  lightingFolder
    .addBinding(config, "pointLightIntensity", { min: 0, max: 2 })
    .on("change", updateLights);
  lightingFolder
    .addBinding(config, "backgroundColor", { view: "color" })
    .on("change", updateBackground);
};

const updateLetterPositions = () => {
  const chars = ["A", "D", "L", "P", "R"];
  const totalWidth = (chars.length - 1) * config.letterSpacing;
  const startX = -totalWidth / 2;

  letters.forEach((letter, index) => {
    letter.position.x = startX + index * config.letterSpacing;
  });
};

const updateLetterSize = () => {
  letters.forEach((letter) => {
    letter.scale.set(config.letterSize, config.letterSize, 1);
  });
};

const updateLights = () => {
  scene.children.forEach((child) => {
    if (child instanceof THREE.AmbientLight) {
      child.intensity = config.ambientLightIntensity;
    }
    if (child instanceof THREE.PointLight) {
      child.intensity = config.pointLightIntensity;
    }
  });
};

const updateBackground = () => {
  scene.background = new THREE.Color(config.backgroundColor);
  renderer.setClearColor(config.backgroundColor);
};

const init = () => {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(config.backgroundColor);
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(config.backgroundColor);
  document.querySelector("#canvas-container")?.appendChild(renderer.domElement);

  const ambientLight = new THREE.AmbientLight(
    0xffffff,
    config.ambientLightIntensity
  );
  scene.add(ambientLight);
  const pointLight = new THREE.PointLight(0xffffff, config.pointLightIntensity);
  pointLight.position.set(
    config.pointLightPosition.x,
    config.pointLightPosition.y,
    config.pointLightPosition.z
  );
  scene.add(pointLight);

  recreateLetters();
  camera.position.z = 5;
  initGui();
};

const recreateLetters = () => {
  // Remove existing letters
  letters.forEach((letter) => scene.remove(letter));
  letters = [];

  const chars = ["A", "D", "L", "P", "R"];
  const totalWidth = (chars.length - 1) * config.letterSpacing;
  const startX = -totalWidth / 2;

  chars.forEach((char, index) => {
    const canvas = document.createElement("canvas");
    const context = canvas.getContext("2d");
    if (context) {
      canvas.width = 128;
      canvas.height = 128;

      context.fillStyle = config.backgroundColor;
      context.fillRect(0, 0, canvas.width, canvas.height);

      context.font = `bold ${config.fontSize}px Arial`;
      context.textAlign = "center";
      context.fillStyle = config.fontColor;
      context.fillText(char, canvas.width / 2, canvas.height / 2 + 20);

      const texture = new THREE.CanvasTexture(canvas);
      const material = new THREE.MeshBasicMaterial({
        map: texture,
        transparent: true,
      });

      const geometry = new THREE.PlaneGeometry(
        config.letterSize,
        config.letterSize
      );
      const letter = new THREE.Mesh(geometry, material);
      letter.position.x = startX + index * config.letterSpacing;
      letters.push(letter);
      scene.add(letter);
    }
  });
};

const animate = () => {
  frameId = requestAnimationFrame(animate);

  letters.forEach((letter, index) => {
    // Rotation based on mouse position with configurable factor
    letter.rotation.x = mouse.y * config.rotationFactor;
    letter.rotation.y = mouse.x * config.rotationFactor;

    // Add hover effect based on mouse distance to letter
    const letterScreenPos = new THREE.Vector3(
      letter.position.x,
      letter.position.y,
      0
    ).project(camera);
    const dist = Math.sqrt(
      Math.pow(letterScreenPos.x - mouse.x, 2) +
        Math.pow(letterScreenPos.y - mouse.y, 2)
    );

    // Push letters away from mouse cursor with configurable strength
    const pushFactor = Math.max(0, 1 - dist) * config.pushStrength;
    letter.position.z = pushFactor;
  });

  renderer.render(scene, camera);
};

const handleMouseMove = (event: MouseEvent) => {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
};

const handleResize = () => {
  if (camera && renderer) {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  }
};

onMounted(() => {
  init();
  animate();
  window.addEventListener("mousemove", handleMouseMove);
  window.addEventListener("resize", handleResize);
});

onBeforeUnmount(() => {
  if (frameId) {
    cancelAnimationFrame(frameId);
  }
  window.removeEventListener("mousemove", handleMouseMove);
  window.removeEventListener("resize", handleResize);
  renderer?.dispose();
});
</script>

<template>
  <div id="canvas-container" class="w-full h-full bg-black"></div>
</template>

<style lang="scss" scoped>
#canvas-container {
  position: relative;
  canvas {
    position: absolute;
    top: 0;
    left: 0;
  }
}
</style>
