<script setup lang="ts">
import { onMounted, onBeforeUnmount, ref } from "vue";
import * as THREE from "three";
import gsap from "gsap";
import { OrbitControls } from "three/addons/controls/OrbitControls.js";

const canvas = ref<HTMLCanvasElement>();
let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let cube: THREE.Mesh;
let controls: OrbitControls;
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
  const materials = [
    new THREE.MeshBasicMaterial({ color: "#0000ff" }), // right (azul)
    new THREE.MeshBasicMaterial({ color: "#ff0000" }), // left (rojo)
    new THREE.MeshBasicMaterial({ color: "#00ff00" }), // top (verde)
    new THREE.MeshBasicMaterial({ color: "#ff69b4" }), // bottom (rosa)
    new THREE.MeshBasicMaterial({ color: "#800080" }), // front (morado)
    new THREE.MeshBasicMaterial({ color: "#ffff00" }), // back (amarillo)
  ];
  cube = new THREE.Mesh(geometry, materials);
  scene.add(cube);

  controls = new OrbitControls(camera, canvas.value!);
  controls.enableDamping = true;

  renderer = new THREE.WebGLRenderer({ canvas: canvas.value });
  renderer.setSize(canvas.value!.clientWidth, canvas.value!.clientHeight);
  animate();
};

const animate = () => {
  // gsap.to(cube.rotation, {
  //   y: cube.rotation.y + Math.PI * 2,
  //   duration: 2,
  //   ease: "none",
  //   repeat: -1,
  // });

  
  const render = () => {
    controls.update();
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
