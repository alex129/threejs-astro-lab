<script setup lang="ts">
import * as THREE from "three";
import { TextGeometry } from "three/addons/geometries/TextGeometry.js";
import { FontLoader } from "three/addons/loaders/FontLoader.js";
import { Pane } from "tweakpane";
import { onMounted, onUnmounted } from "vue";

// Scene setup
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let pane: Pane;

// Configuration
const config = {
  text: "Text example",
  fontSize: 1,
  letterSpacing: 0.1,
  mouseRadius: 2,
  mouseStrength: 1,
  returnSpeed: 1,
  physicsStrength: 0.1,
  letterSize: 0.5,
};

// Mouse position
const mouse = new THREE.Vector2();
const mouse3D = new THREE.Vector3();

// Letter objects
let letters: THREE.Mesh[] = [];
let originalPositions: THREE.Vector3[] = [];
let velocities: THREE.Vector3[] = [];

// Initialize Tweakpane
const initTweakpane = () => {
  pane = new Pane();
  const textFolder = pane.addFolder({ title: "Text Settings" });
  textFolder.addBinding(config, "text");
  textFolder.addBinding(config, "fontSize", { min: 0.5, max: 3, step: 0.1 });
  textFolder.addBinding(config, "letterSpacing", {
    min: 0,
    max: 0.5,
    step: 0.01,
  });
  textFolder.addBinding(config, "letterSize", { min: 0.1, max: 1, step: 0.1 });

  const physicsFolder = pane.addFolder({ title: "Physics Settings" });
  physicsFolder.addBinding(config, "mouseRadius", {
    min: 0.5,
    max: 5,
    step: 0.1,
  });
  physicsFolder.addBinding(config, "mouseStrength", {
    min: 0.1,
    max: 2,
    step: 0.1,
  });
  physicsFolder.addBinding(config, "returnSpeed", {
    min: 0.1,
    max: 2,
    step: 0.1,
  });
  physicsFolder.addBinding(config, "physicsStrength", {
    min: 0.01,
    max: 0.5,
    step: 0.01,
  });

  pane.on("change", () => {
    createText();
  });
};

// Setup scene
const setupScene = () => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  renderer = new THREE.WebGLRenderer({
    canvas: document.querySelector("#canvas") as HTMLCanvasElement,
    antialias: true,
    alpha: true,
  });

  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  camera.position.z = 5;
};

// Create text geometry
const createText = () => {
  // Clear existing letters
  letters.forEach((letter) => scene.remove(letter));
  letters = [];
  originalPositions = [];
  velocities = [];

  const loader = new FontLoader();
  loader.load("/fonts/helvetiker_regular.typeface.json", (font) => {
    const textGeometry = new TextGeometry(config.text, {
      font: font,
      size: config.fontSize,
      curveSegments: 12,
      bevelEnabled: false,
    });

    textGeometry.computeBoundingBox();
    const centerOffset =
      -(textGeometry.boundingBox!.max.x - textGeometry.boundingBox!.min.x) / 2;

    let currentX = centerOffset;
    for (let i = 0; i < config.text.length; i++) {
      // Create shape for single letter
      const letterShape = font.generateShapes(config.text[i], config.fontSize);
      const letterGeometry = new THREE.ShapeGeometry(letterShape);
      const letterMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff });

      const letter = new THREE.Mesh(letterGeometry, letterMaterial);
      letter.position.x = currentX;
      letter.position.y = 0;
      letter.scale.set(config.letterSize, config.letterSize, 1);

      scene.add(letter);
      letters.push(letter);
      originalPositions.push(letter.position.clone());
      velocities.push(new THREE.Vector3());

      currentX += config.letterSpacing + config.fontSize * 0.6;
    }
  });
};

// Mouse move handler
const onMouseMove = (event: MouseEvent) => {
  mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;

  mouse3D.set(mouse.x * 5, mouse.y * 5, 0);
};

// Window resize handler
const onResize = () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
};

// Animation loop
const animate = () => {
  requestAnimationFrame(animate);

  // Update letter positions based on mouse proximity
  letters.forEach((letter, i) => {
    const distance = letter.position.distanceTo(mouse3D);

    if (distance < config.mouseRadius) {
      const force = new THREE.Vector3()
        .subVectors(letter.position, mouse3D)
        .normalize()
        .multiplyScalar(
          config.mouseStrength * (1 - distance / config.mouseRadius)
        );

      velocities[i].add(force.multiplyScalar(config.physicsStrength));
    }

    // Apply velocity and damping
    letter.position.add(velocities[i]);
    velocities[i].multiplyScalar(0.95);

    // Return to original position
    const returnForce = new THREE.Vector3()
      .subVectors(originalPositions[i], letter.position)
      .multiplyScalar(config.returnSpeed * 0.01);

    velocities[i].add(returnForce);
  });

  renderer.render(scene, camera);
};

// Initialize everything
const init = () => {
  setupScene();
  initTweakpane();
  createText();
  window.addEventListener("mousemove", onMouseMove);
  window.addEventListener("resize", onResize);
  animate();
};

// Cleanup
const cleanup = () => {
  window.removeEventListener("mousemove", onMouseMove);
  window.removeEventListener("resize", onResize);
  pane.dispose();
};

onMounted(() => {
  init();
});

onUnmounted(() => {
  cleanup();
});
</script>

<template>
  <div class="w-full h-screen bg-black">
    <canvas id="canvas" class="w-full h-full"></canvas>
  </div>
</template>

<style scoped>
canvas {
  touch-action: none;
}
</style>
