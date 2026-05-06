<template>
  <div class="app-container">
    <iframe src="/globe.html" class="globe-iframe" title="Interactive 3D Globe"> </iframe>

    <div class="dashboard-wrapper">
      <Dashboard :projectId="selectedProject" @close="selectedProject = undefined" />
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, onUnmounted } from 'vue'
import Dashboard from './components/Dashboard.vue'

const selectedProject = ref<string | undefined>(undefined)

const handleMessage = (event: MessageEvent) => {
  if (event.data && event.data.type === 'REGION_CLICK') {
    selectedProject.value = event.data.regionId
  }
}

onMounted(() => {
  window.addEventListener('message', handleMessage)
})

onUnmounted(() => {
  window.removeEventListener('message', handleMessage)
})
</script>

<!-- STYLE GLOBAL (Sans le mot-clé scoped) -->
<style>
body {
  margin: 0;
  padding: 0;
  overflow: hidden;
  background-color: #040810;
}

* {
  box-sizing: border-box;
}
</style>

<!-- STYLE LOCAL (Appliqué uniquement à ce composant) -->
<style scoped>
.app-container {
  position: relative;
  width: 100vw;
  height: 100vh;
  overflow: hidden;
}

.globe-iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: none;
  z-index: 1; /* S'assure que le globe reste en dessous */
}

.dashboard-wrapper {
  position: absolute;
  top: 50%;
  right: 30px;
  transform: translateY(-50%);
  width: 420px;
  max-width: calc(100vw - 60px);
  height: 85vh;
  max-height: 800px;
  z-index: 50;
  display: flex;
}
</style>
