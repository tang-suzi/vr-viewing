<template>
  <div class="container" ref="container"></div>
  <div class="map">
    <div class="tag" ref="tagDiv"></div>
    <img src="@/assets/map.gif" alt="" />
  </div>
  <div class="loading" v-if="progress < 100"></div>
  <div class="progress" v-if="progress < 100">
    <img src="@/assets/loading.gif" alt="Loading..." />
    <span>新房奔跑中：{{ progress }}%</span>
  </div>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { ref, onMounted } from "vue";
import gsap from "gsap";

const tagDiv = ref(null);
const progress = ref(0);
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(
  75,
  window.innerWidth / window.innerHeight,
  0.1,
  1000
);
camera.position.set(0, 0, 0);

const renderer = new THREE.WebGLRenderer({
  antialias: true,
  alpha: true,
  logarithmicDepthBuffer: true,
});
renderer.setSize(window.innerWidth, window.innerHeight);

const container = ref(null);

onMounted(() => {
  container.value.appendChild(renderer.domElement);
  animate();
  setupMouseControls();
  setupRooms();
});

function animate() {
  renderer.render(scene, camera);
  requestAnimationFrame(animate);
}

function setupMouseControls() {
  let isMouseDown = false;
  let clock = new THREE.Clock();

  const onMouseMove = (event) => {
    camera.rotation.order = "YXZ";
    const delta = clock.getDelta();
    if (isMouseDown) {
      gsap.to(camera.rotation, {
        y: camera.rotation.y + event.movementX * 0.002,
        x: camera.rotation.x + event.movementY * 0.002,
        duration: delta,
      });
    }
  };

  container.value.addEventListener("mousedown", () => (isMouseDown = true));
  container.value.addEventListener("mouseup", () => (isMouseDown = false));
  container.value.addEventListener("mouseout", () => (isMouseDown = false));
  container.value.addEventListener("mousemove", onMouseMove);
}

function moveTag(name) {
  const positions = {
    客厅: [100, 110],
    厨房: [180, 190],
    阳台: [50, 50],
  };
  if (positions[name]) {
    gsap.to(tagDiv.value, {
      duration: 0.5,
      x: positions[name][0],
      y: positions[name][1],
      ease: "power3.inOut",
    });
  }
}

function setupRooms() {
  const rooms = [
    new Room("客厅", 0, "./../assets/img/livingroom/"),
    new Room(
      "厨房",
      3,
      "./../assets/img/kitchen/",
      new THREE.Vector3(-5, 0, -10),
      new THREE.Euler(0, -Math.PI / 2, 0)
    ),
    new Room(
      "阳台",
      8,
      "./../assets/img/balcony/",
      new THREE.Vector3(0, 0, 15)
    ),
  ];

  const sprites = [
    new SpriteText("厨房", new THREE.Vector3(-1, 0, -3), () =>
      moveTo("厨房", new THREE.Vector3(-5, 0, -10))
    ),
    new SpriteText("客厅", new THREE.Vector3(-4, 0, -6), () =>
      moveTo("客厅", new THREE.Vector3(0, 0, 0))
    ),
    new SpriteText("阳台", new THREE.Vector3(0, 0, 3), () =>
      moveTo("阳台", new THREE.Vector3(0, 0, 15))
    ),
    new SpriteText("客厅", new THREE.Vector3(-1, 0, 11), () =>
      moveTo("客厅", new THREE.Vector3(0, 0, 0))
    ),
  ];
}

function moveTo(name, position) {
  gsap.to(camera.position, {
    duration: 1,
    ...position,
  });
  moveTag(name);
}

class Room {
  constructor(
    name,
    index,
    texturePath,
    position = new THREE.Vector3(),
    euler = new THREE.Euler()
  ) {
    const geometry = new THREE.BoxGeometry(10, 10, 10).scale(1, 1, -1);
    const faces = ["l", "r", "u", "d", "b", "f"].map((f) => `${index}_${f}`);

    const materials = faces.map((face) => {
      const tex = new THREE.TextureLoader().load(
        new URL(`${texturePath}${face}.jpg`, import.meta.url).href
      );
      if (["u", "d"].includes(face.split("_")[1])) {
        tex.rotation = Math.PI;
        tex.center = new THREE.Vector2(0.5, 0.5);
      }
      return new THREE.MeshBasicMaterial({ map: tex });
    });

    const mesh = new THREE.Mesh(geometry, materials);
    mesh.position.copy(position);
    mesh.rotation.copy(euler);
    scene.add(mesh);
  }
}

class SpriteText {
  constructor(text, position, onClick) {
    const canvas = document.createElement("canvas");
    canvas.width = canvas.height = 1024;
    const ctx = canvas.getContext("2d");
    ctx.fillStyle = "rgba(100, 100, 100, 0.7)";
    ctx.fillRect(0, 256, 1024, 512);
    ctx.font = "bold 200px Arial";
    ctx.fillStyle = "white";
    ctx.textAlign = "center";
    ctx.textBaseline = "middle";
    ctx.fillText(text, 512, 512);

    const material = new THREE.SpriteMaterial({
      map: new THREE.CanvasTexture(canvas),
      transparent: true,
      depthWrite: true,
    });

    const sprite = new THREE.Sprite(material);
    sprite.scale.set(0.5, 0.5, 0.5);
    sprite.position.copy(position);
    sprite.renderOrder = 1;
    scene.add(sprite);

    const raycaster = new THREE.Raycaster();
    const mouse = new THREE.Vector2();

    window.addEventListener("click", (event) => {
      mouse.x = (event.clientX / window.innerWidth) * 2 - 1;
      mouse.y = -(event.clientY / window.innerHeight) * 2 + 1;
      raycaster.setFromCamera(mouse, camera);
      if (
        raycaster.intersectObject(sprite).length > 0 &&
        typeof onClick === "function"
      ) {
        onClick();
      }
    });
  }
}

THREE.DefaultLoadingManager.onProgress = (item, loaded, total) => {
  progress.value = ((loaded / total) * 100).toFixed(2);
};
</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
.container {
  height: 100vh;
  width: 100vw;
  background-color: #f0f0f0;
}
.map {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 300px;
  height: 260px;
  overflow: hidden;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.5);
}
.map > img {
  width: 100%;
  height: 100%;
  position: absolute;
  left: 0;
  top: 0;
}
.tag {
  position: absolute;
  width: 30px;
  height: 30px;
  background-image: url(@/assets/location.png);
  background-size: cover;
  z-index: 1;
}
.loading {
  position: fixed;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  background-image: url(@/assets/loading.png);
  background-size: cover;
  filter: blur(50px);
  z-index: 100;
}
.progress {
  position: absolute;
  top: 0;
  left: 0;
  width: 100vw;
  height: 100vh;
  z-index: 101;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 20px;
  color: #fff;
}
.progress > img {
  padding: 0 15px;
}
</style>
