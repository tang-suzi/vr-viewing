<template>
  <div ref="containerDiv"></div>
</template>
<script setup>
import * as THREE from "three";
import { ref, onMounted } from "vue";
const containerDiv = ref(null);
let scene, camera, renderer, axesHelper;
const init = () => {
  scene = new THREE.Scene();
  camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );
  renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  containerDiv.value.appendChild(renderer.domElement);

  camera.position.set(0, 0, 5);
  axesHelper = new THREE.AxesHelper(5);
  scene.add(axesHelper);
};
const createBox = () => {
  const geometry = new THREE.BoxGeometry(10, 10, 10);
  geometry.scale(1, 1, -1);
  const material = new THREE.MeshBasicMaterial({ color: 0x003333 });
  const cube = new THREE.Mesh(geometry, material);
  scene.add(cube);
};
const animate = () => {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
};
onMounted(() => {
  init();
  createBox();
  animate();
});
</script>
