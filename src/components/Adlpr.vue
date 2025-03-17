<script setup lang="ts">
import { onMounted, onBeforeUnmount } from "vue";
import * as THREE from "three";

let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let text: THREE.Mesh;
let frameId: number;
let mouse = new THREE.Vector2();

const init = () => {
  scene = new THREE.Scene();
  scene.background = new THREE.Color(0x000000);
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: false });
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setClearColor(0x000000);
  document.querySelector("#canvas-container")?.appendChild(renderer.domElement);

  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  const pointLight = new THREE.PointLight(0xffffff, 1);
  pointLight.position.set(5, 5, 5);
  scene.add(pointLight);

  const canvas = document.createElement("canvas");
  const context = canvas.getContext("2d");
  if (context) {
    canvas.width = 256;
    canvas.height = 128;

    context.fillStyle = "#000000";
    context.fillRect(0, 0, canvas.width, canvas.height);

    context.font = "bold 60px Arial";
    context.textAlign = "center";
    context.fillStyle = "#ffffff";
    context.fillText("ADLPR", canvas.width / 2, canvas.height / 2);

    const texture = new THREE.CanvasTexture(canvas);
    const material = new THREE.MeshBasicMaterial({
      map: texture,
      transparent: true,
    });

    const geometry = new THREE.PlaneGeometry(2, 1);
    text = new THREE.Mesh(geometry, material);
    scene.add(text);
  }

  camera.position.z = 5;
};

const animate = () => {
  frameId = requestAnimationFrame(animate);

  if (text) {
    text.rotation.x = mouse.y * 0.5;
    text.rotation.y = mouse.x * 0.5;

    text.position.y = Math.sin(Date.now() * 0.001) * 0.1;
  }

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
