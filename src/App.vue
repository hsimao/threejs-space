<template>
  <div class="container">
    <div class="screen" ref="screenDom" />

    <div class="pages" ref="pages">
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
import { ref, onMounted } from "vue";

const screenDom = ref(null);

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

  // 星空背景
  const url = require("./assets/images/bg.jpeg");

  // 紋理
  const envTexture = new THREE.TextureLoader().load(url);
  scene.background = envTexture;
  scene.environment = envTexture;

  // 渲染
  const loopRender = () => {
    requestAnimationFrame(loopRender);
    renderer.render(scene, camera);
  };
  loopRender();
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
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
}
</style>
