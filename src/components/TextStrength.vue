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

const config = {
  hoverRange: 0.5,
  textStrength: 1.0,
  letterSpacing: 0.6,
  letterSize: 0.8,
  fontSize: 80,
  textColor: "#ffffff",
  backgroundColor: "#000000",
  hoverColor: "#ff0000"
};

const initGui = () => {
  const pane = new Pane();

  const textFolder = pane.addFolder({ title: "Text" });
  textFolder.addBinding(config, "hoverRange", { min: 0.1, max: 2 });
  textFolder.addBinding(config, "textStrength", { min: 0.1, max: 3 });
  textFolder.addBinding(config, "letterSpacing", { min: 0.1, max: 2 })
    .on("change", updateLetterPositions);
  textFolder.addBinding(config, "letterSize", { min: 0.1, max: 2 })
    .on("change", updateLetterSize);
  textFolder.addBinding(config, "fontSize", { min: 20, max: 120 })
    .on("change", recreateLetters);
  textFolder.addBinding(config, "textColor", { view: "color" })
    .on("change", recreateLetters);
  textFolder.addBinding(config, "hoverColor", { view: "color" })
    .on("change", recreateLetters);
  textFolder.addBinding(config, "backgroundColor", { view: "color" })
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

  recreateLetters();
  camera.position.z = 5;
  initGui();
};

const recreateLetters = () => {
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
      context.fillStyle = config.textColor;
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

  letters.forEach((letter) => {
    const letterScreenPos = new THREE.Vector3(
      letter.position.x,
      letter.position.y,
      0
    ).project(camera);
    
    const dist = Math.sqrt(
      Math.pow(letterScreenPos.x - mouse.x, 2) +
      Math.pow(letterScreenPos.y - mouse.y, 2)
    );

    if (dist < config.hoverRange) {
      const strength = (1 - dist / config.hoverRange) * config.textStrength;
      letter.scale.set(1 + strength, 1 + strength, 1);
      (letter.material as THREE.MeshBasicMaterial).color = new THREE.Color(config.hoverColor);
    } else {
      letter.scale.set(1, 1, 1);
      (letter.material as THREE.MeshBasicMaterial).color = new THREE.Color(config.textColor);
    }
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
