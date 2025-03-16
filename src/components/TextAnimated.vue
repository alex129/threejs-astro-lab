<script setup lang="ts">
import { onMounted, onBeforeUnmount } from "vue";
import * as THREE from "three";
import { FontLoader } from "three/examples/jsm/loaders/FontLoader.js";
import { TextGeometry } from "three/examples/jsm/geometries/TextGeometry.js";

let scene: THREE.Scene;
let camera: THREE.PerspectiveCamera;
let renderer: THREE.WebGLRenderer;
let text: THREE.Mesh;
let frameId: number;
let mouse = new THREE.Vector2();

const init = () => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true });
  renderer.setSize(window.innerWidth, window.innerHeight);
  document.querySelector("#canvas-container")?.appendChild(renderer.domElement);

  // Add lights
  const ambientLight = new THREE.AmbientLight(0xffffff, 0.5);
  scene.add(ambientLight);
  const pointLight = new THREE.PointLight(0xffffff, 1);
  pointLight.position.set(5, 5, 5);
  scene.add(pointLight);

  // Create 3D text
  const loader = new FontLoader();
  loader.load(
    "https://fonts.googleapis.com/css2?family=Cairo:wght@200..1000&family=Open+Sans:ital,wght@0,300..800;1,300..800&family=Roboto:ital,wght@0,100..900;1,100..900&display=swap∏",
    function (font) {
      const textGeometry = new TextGeometry("ADLPR", {
        font: font,
        size: 0.5,
        curveSegments: 12,
        bevelEnabled: true,
        bevelThickness: 0.03,
        bevelSize: 0.02,
        bevelOffset: 0,
        bevelSegments: 5,
      });

      const material = new THREE.MeshPhongMaterial({
        color: 0x00ff00,
        specular: 0x555555,
        shininess: 30,
      });

      text = new THREE.Mesh(textGeometry, material);

      // Center the text
      textGeometry.computeBoundingBox();
      const textWidth =
        textGeometry.boundingBox!.max.x - textGeometry.boundingBox!.min.x;
      text.position.x = -textWidth / 2;

      scene.add(text);
    }
  );

  camera.position.z = 5;
};

const animate = () => {
  frameId = requestAnimationFrame(animate);

  if (text) {
    // Apply effects based on mouse position
    text.rotation.x = mouse.y * 0.5;
    text.rotation.y = mouse.x * 0.5;

    // Add some floating animation
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
  <div id="canvas-container" class="w-full h-full"></div>
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
