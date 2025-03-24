<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref } from "vue";
import * as THREE from "three";

const canvas = ref<HTMLCanvasElement>();
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let cube: THREE.Mesh;
let animationFrameId: number;

const init = () => {
  scene = new THREE.Scene();
  scene.background = new THREE.Color("#000000");

  camera = new THREE.PerspectiveCamera(
    75,
    canvas.value!.clientWidth / canvas.value!.clientHeight
  );
  camera.position.z = 3;
  scene.add(camera);

  const geometry = new THREE.BoxGeometry(1, 1, 1);
  const material = new THREE.MeshBasicMaterial({ color: "#00ff00" });
  cube = new THREE.Mesh(geometry, material);
  scene.add(cube);

  renderer = new THREE.WebGLRenderer({ canvas: canvas.value });
  renderer.setSize(canvas.value!.clientWidth, canvas.value!.clientHeight);
  animate();
};

const animate = () => {
  cube.rotation.y += 0.01;
  renderer.render(scene, camera);
  animationFrameId = requestAnimationFrame(animate);
};

onMounted(() => {
  init();
});

onBeforeUnmount(() => {
  cancelAnimationFrame(animationFrameId);
  renderer?.dispose();
});
</script>

<template>
  <canvas ref="canvas" class="w-full h-full bg-black"></canvas>
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
