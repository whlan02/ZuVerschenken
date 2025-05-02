<script setup>
import { ref, onMounted, computed } from 'vue';
import MapView from './components/MapView.vue'

const mapViewRef = ref(null);

// Fenstergrößenänderung überwachen
onMounted(() => {
  window.addEventListener('resize', () => {
    windowWidth.value = window.innerWidth;
  });
});

// Überprüfen, ob es sich um ein mobiles Gerät handelt
const isMobile = computed(() => {
  // Eine genauere Methode zur Erkennung mobiler Geräte verwenden
  const isTouchDevice = 'ontouchstart' in window || navigator.maxTouchPoints > 0;
  return isTouchDevice && window.innerWidth <= 768;
});
</script>

<template>
  <div class="app-container">
    <MapView ref="mapViewRef" />
  </div>
</template>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html, body, #app, .app-container {
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}
</style>
