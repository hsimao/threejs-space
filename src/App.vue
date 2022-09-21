<template>
  <div class="container">
    <div class="screen" ref="screenDom" />

    <div class="pages" ref="pagesDom">
      <div class="page">
        <h2 class="title">page 1</h2>
        <p>desc...</p>
      </div>

      <div class="page">
        <h2 class="title">page 2</h2>
        <p>desc...</p>
      </div>

      <div class="page">
        <h2 class="title">page 3</h2>
        <p>desc...</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import * as THREE from "three";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import gsap from "gsap";

import { ref, onMounted } from "vue";

const screenDom = ref(null);
const pagesDom = ref(null);

const handleLoadModelWrapper = (gltf, scene, optionsFn) => {
  if (typeof optionsFn === "function") optionsFn(gltf);

  scene.add(gltf.scene);

  // 滑鼠控制模型
  window.addEventListener("mousemove", (e) => handleMousemove(gltf, e));
};

let currentPage = 0;
const updatePageByScroll = (e) => {
  // 往下滾
  if (e.wheelDelta < 0) {
    currentPage++;
    if (currentPage > 2) currentPage = 2;
  }

  // 往上滾
  if (e.wheelDelta > 0) {
    currentPage--;
    if (currentPage < 0) currentPage = 0;
  }

  return currentPage;
};

// handle scroll
const scrollTimeline = gsap.timeline();
const handleMousewheel = (camera, e) => {
  if (scrollTimeline.isActive()) return;

  // update page
  const page = updatePageByScroll(e);

  if (!scrollTimeline.isActive()) {
    // update camera position
    const cameraOffsetY = page * -8;
    scrollTimeline.to(camera.position, { y: cameraOffsetY, duration: 1 });

    // update pages position
    const domOffsetY = -page * window.innerHeight;
    updatePagesPosition(domOffsetY);
  }
};

// handle mousemove
const handleMousemove = (gltf, e) => {
  // 算出當前鼠標對應中心點的偏移比例
  const x = (e.clientX / window.innerWidth) * 2 - 1;
  const y = (e.clientY / window.innerHeight) * 2 - 1;

  // update rotate
  updateModelRotate(gltf, { x: y, y: x, duration: 1 });
};

// 更新 model rotate 位置
const updateModelRotate = (gltf, { x, y, duration }) => {
  const timeline = gsap.timeline();
  timeline.to(gltf.scene.rotation, {
    duration,
    x,
    y
  });
};

// update pages div
const updatePagesPosition = (y) => gsap.to(pagesDom.value, { duration: 1, y });

// mounted
onMounted(() => {
  // 場景
  const scene = new THREE.Scene();

  // 相機
  const camera = new THREE.PerspectiveCamera(
    45,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  camera.position.set(0, 0, 10);

  // 渲染器
  const renderer = new THREE.WebGLRenderer({ antialias: true });
  renderer.setSize(window.innerWidth, window.innerHeight);

  // 渲染到 div
  screenDom.value.appendChild(renderer.domElement);

  // 星空背景圖
  const url = require("./assets/images/bg.jpeg");

  // 紋理
  const envTexture = new THREE.TextureLoader().load(url);

  // 球面紋理
  envTexture.mapping = THREE.EquirectangularReflectionMapping;

  scene.background = envTexture;
  scene.environment = envTexture;

  // 渲染
  const loopRender = () => {
    requestAnimationFrame(loopRender);
    renderer.render(scene, camera);
  };
  loopRender();

  // 燈光
  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(0, 0, 1);
  scene.add(light);

  const light2 = new THREE.DirectionalLight(0xffffff, 0.5);
  light2.position.set(0, 0, -1);
  scene.add(light2);

  const light3 = new THREE.DirectionalLight(0xffffff, 0.5);
  light3.position.set(-1, 1, 1);
  scene.add(light3);

  // 解壓縮的加載器
  const dracoLoader = new DRACOLoader();
  dracoLoader.setDecoderPath("./draco/gltf/");
  dracoLoader.setDecoderConfig({ type: "js" });

  const loader = new GLTFLoader();
  loader.setDRACOLoader(dracoLoader);

  // 載入 太空梭 3d 模型 1
  loader.load("./model/xz.glb", (gltf) =>
    handleLoadModelWrapper(gltf, scene, (gltf) => {
      // 縮小
      gltf.scene.scale.set(0.1, 0.1, 0.1);
      // 向右偏移
      gltf.scene.position.set(3, 0, 0);
    })
  );

  // 載入 太空梭 3d 模型 2
  loader.load("./model/xq6.glb", (gltf) =>
    handleLoadModelWrapper(gltf, scene, (gltf) => {
      // 縮小
      gltf.scene.scale.set(0.1, 0.1, 0.1);
      // 向右, 向下偏移
      gltf.scene.position.set(3, -8, 0);
    })
  );

  // 載入 太空梭 3d 模型 3
  loader.load("./model/gr75.glb", (gltf) =>
    handleLoadModelWrapper(gltf, scene, (gltf) => {
      // 向右, 向下偏移
      gltf.scene.position.set(3, -16, 0);
    })
  );

  // 載入月亮 3d 模式, 來當成星星
  loader.load("./model/moon.glb", (gltf) => {
    const moon = gltf.scene.children[0];
    scene.add(moon);
  });

  // 滑鼠滾動控制 camera
  window.addEventListener("mousewheel", (e) => handleMousewheel(camera, e));
});
</script>

<style>
*,
*:before,
*:after {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background-color: #000;
}

.container {
  width: 100vw;
  height: 100vh;
  transform-origin: 0 0;
  position: relative;
}

.screen {
  width: 100vw;
  height: 100vh;
}

.pages {
  position: fixed;
  top: 0;
  left: 0;
}

.pages .page {
  width: 100vw;
  height: 100vh;
  justify-content: center;
  align-items: flex-start;
  color: #fff;
  padding: 250px;
  box-sizing: border-box;
}

.pages .title {
  font-size: 50px;
  font-weight: 900;
  margin-bottom: 20px;
}

.pages p {
  font-size: 25px;
}
</style>
