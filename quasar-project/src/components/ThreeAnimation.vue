<template>
    <div ref="threeContainer" class="three-container"></div>
  </template>
  
  <script setup>
  import { onMounted, ref } from 'vue';
  import * as THREE from 'three';
  
  const threeContainer = ref(null);
  
  onMounted(() => {
    let camera, scene, renderer, mesh;
  
    function init() {
      const width = threeContainer.value.clientWidth;
      const height = threeContainer.value.clientHeight;
  
      camera = new THREE.PerspectiveCamera(70, width / height, 0.1, 100);
      camera.position.z = 2;
  
      scene = new THREE.Scene();
  
      const texture = new THREE.TextureLoader().load('textures/crate.gif');
      texture.colorSpace = THREE.SRGBColorSpace;
  
      const geometry = new THREE.BoxGeometry();
      const material = new THREE.MeshBasicMaterial({ color: 0x00ff00 }); // Color verde

  
      mesh = new THREE.Mesh(geometry, material);
      scene.add(mesh);
  
      renderer = new THREE.WebGLRenderer({ antialias: true });
      renderer.setPixelRatio(window.devicePixelRatio);
      renderer.setSize(width, height);
      renderer.setAnimationLoop(animate);
  
      threeContainer.value.appendChild(renderer.domElement);
  
      window.addEventListener('resize', onWindowResize);
    }
  
    function onWindowResize() {
      const width = threeContainer.value.clientWidth;
      const height = threeContainer.value.clientHeight;
  
      camera.aspect = width / height;
      camera.updateProjectionMatrix();
  
      renderer.setSize(width, height);
    }
  
    function animate() {
      mesh.rotation.x += 0.005;
      mesh.rotation.y += 0.01;
  
      renderer.render(scene, camera);
    }
  
    init();
  });
  </script>
  
  <style scoped>
  .three-container {
    width: 100%;
    height: 100vh; /* Ajusta esto para que el contenedor ocupe toda la ventana */
    overflow: hidden; /* Asegúrate de que no haya desplazamiento */
  }
  </style>
  