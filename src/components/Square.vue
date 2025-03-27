<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref } from "vue";
import * as THREE from "three";
import gsap from "gsap";

const canvas = ref<HTMLCanvasElement>();
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let cube: THREE.Mesh;

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
  gsap.to(cube.rotation, {
    y: cube.rotation.y + Math.PI * 0.5,
    duration: 2,
    ease: "none",
    repeat: -1,
  });

  const render = () => {
    renderer.render(scene, camera);
    requestAnimationFrame(render);
  };
  render();
};

onMounted(() => {
  init();
});

onBeforeUnmount(() => {
  gsap.killTweensOf(cube.rotation);
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
