<template>
  <div class="dashboard-panel">
    <!-- ÉTAT VIDE : Aucun projet sélectionné -->
    <div v-if="!projectId" class="empty-state">
      <div class="ping-animation"></div>
      <h3 class="empty-title">En attente de sélection</h3>
      <p class="empty-desc">
        Tournez le globe et cliquez sur un marqueur vert pour générer le rapport d'analyse
        d'investissement.
      </p>
    </div>

    <!-- ÉTAT ACTIF : Projet sélectionné -->
    <div v-else class="active-state">
      <button class="close-btn" @click="$emit('close')">✕</button>

      <header class="dashboard-header">
        <div class="region-label">Rapport d'Analyse</div>
        <h2 class="region-name">{{ projectId }}</h2>
      </header>

      <nav class="dashboard-menu">
        <button
          v-for="tab in tabs"
          :key="tab.id"
          class="ctrl-btn"
          :class="{ active: activeTab === tab.id }"
          @click="activeTab = tab.id"
        >
          {{ tab.label }}
        </button>
      </nav>

      <div class="dashboard-content">
        <div v-if="activeTab === 'overview'" class="tab-pane fade-in">
          <div class="chart-paper"><RiskFlagsChart :data="flagsData" /></div>
          <div class="chart-paper"><PillarsRadarChart :data="radarData" /></div>
        </div>
        <div v-if="activeTab === 'risks'" class="tab-pane fade-in">
          <div class="chart-paper"><DivergingBarChart :data="findingsData" /></div>
        </div>
        <div v-if="activeTab === 'climate'" class="tab-pane fade-in">
          <div class="chart-paper"><CarbonImpact :data="climateData" /></div>
        </div>
        <div v-if="activeTab === 'ecosystem'" class="tab-pane fade-in">
          <div class="chart-paper"><StakeholderNetwork :data="networkData" /></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

import DivergingBarChart from './DivergingBarChart.vue'
import CarbonImpact from './CarbonImpact.vue'
import RiskFlagsChart from './RiskFlagsChart.vue'
import PillarsRadarChart from './PillarsRadarChart.vue'
import StakeholderNetwork from './StakeholderNetwork.vue'

defineOptions({ name: 'DashboardView' })

defineProps({
  projectId: { type: String, default: null }, // Par défaut, null
})

defineEmits(['close'])

const activeTab = ref('overview')
const tabs = [
  { id: 'overview', label: 'Overview' },
  { id: 'risks', label: 'Risks & Benefits' },
  { id: 'climate', label: 'Climate' },
  { id: 'ecosystem', label: 'Ecosystem' },
]

// --- DONNÉES TEMPORAIRES ---
const findingsData = ref([
  { category: 'Design', positive: 42, negative: -45 },
  { category: 'Governance', positive: 47, negative: -49 },
  { category: 'Financial', positive: 52, negative: -55 },
  { category: 'Impact', positive: 57, negative: -59 },
  { category: 'Safeguards', positive: 62, negative: -63 },
])
const climateData = ref([
  { year: 2023, bau: 1000, project: 1000 },
  { year: 2025, bau: 1200, project: 900 },
  { year: 2030, bau: 1500, project: 600 },
])
const flagsData = ref({ green: 12, blue: 4, yellow: 2, red: 0 })
const radarData = ref([
  { axis: 'Design', value: 90 },
  { axis: 'Governance', value: 79 },
  { axis: 'Financial', value: 86 },
  { axis: 'Impact', value: 95 },
  { axis: 'Safeguards', value: 98 },
  { axis: 'Growth', value: 97 },
])
const networkData = ref({
  nodes: [
    { id: 'Projet', group: 1 },
    { id: 'Gov', group: 2 },
    { id: 'SDG 13', group: 3 },
  ],
  links: [
    { source: 'Projet', target: 'Gov' },
    { source: 'Projet', target: 'SDG 13' },
  ],
})
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=Syne:wght@400;600&display=swap');

/* Panneau Global */
.dashboard-panel {
  background: rgba(4, 12, 24, 0.85);
  border: 0.5px solid rgba(0, 212, 255, 0.25);
  border-radius: 8px;
  backdrop-filter: blur(12px);
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.5);

  width: 100%;
  height: 100%;
  box-sizing: border-box;

  display: flex;
  flex-direction: column;
  overflow: hidden;
}

/* --- ÉTAT VIDE (Empty State) --- */
.empty-state {
  flex: 1;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 40px;
  text-align: center;
}

.ping-animation {
  width: 20px;
  height: 20px;
  background-color: #5fba53;
  border-radius: 50%;
  margin-bottom: 24px;
  box-shadow: 0 0 0 0 rgba(95, 186, 83, 0.7);
  animation: pulse-ping 2s infinite cubic-bezier(0.66, 0, 0, 1);
}

@keyframes pulse-ping {
  to {
    box-shadow: 0 0 0 30px rgba(95, 186, 83, 0);
  }
}

.empty-title {
  font-family: 'Space Mono', monospace;
  font-size: 12px;
  letter-spacing: 0.15em;
  color: #5fba53;
  text-transform: uppercase;
  margin-bottom: 12px;
}

.empty-desc {
  font-family: 'Syne', sans-serif;
  font-size: 14px;
  color: rgba(232, 244, 255, 0.6);
  line-height: 1.6;
}

/* --- ÉTAT ACTIF --- */
.active-state {
  display: flex;
  flex-direction: column;
  height: 100%;
}

.dashboard-header {
  padding: 20px 24px 10px 24px;
  position: relative;
}
.region-label {
  font-family: 'Space Mono', monospace;
  font-size: 9px;
  letter-spacing: 0.2em;
  color: #5fba53;
  text-transform: uppercase;
  margin-bottom: 8px;
}
.region-name {
  font-family: 'Syne', sans-serif;
  font-size: 26px;
  font-weight: 600;
  color: #e8f4ff;
  margin: 0;
}

.close-btn {
  position: absolute;
  top: 15px;
  right: 15px;
  background: none;
  border: none;
  color: rgba(232, 244, 255, 0.45);
  font-size: 18px;
  cursor: pointer;
}
.close-btn:hover {
  color: #e8f4ff;
}

.dashboard-menu {
  display: flex;
  gap: 8px;
  padding: 10px 24px;
  border-bottom: 0.5px solid rgba(0, 212, 255, 0.15);
  overflow-x: auto;
}
.dashboard-menu::-webkit-scrollbar {
  display: none;
}
.ctrl-btn {
  font-family: 'Space Mono', monospace;
  font-size: 9px;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  background: rgba(4, 12, 28, 0.8);
  border: 0.5px solid rgba(0, 212, 255, 0.35);
  color: #e8f4ff;
  padding: 8px 12px;
  border-radius: 2px;
  cursor: pointer;
  transition: all 0.2s;
  white-space: nowrap;
}
.ctrl-btn:hover {
  background: rgba(0, 212, 255, 0.12);
  border-color: #5fba53;
  color: #5fba53;
}
.ctrl-btn.active {
  background: rgba(0, 212, 255, 0.15);
  border-color: #5fba53;
  color: #5fba53;
}

.dashboard-content {
  flex: 1;
  padding: 24px;
  overflow-y: auto;
}
.dashboard-content::-webkit-scrollbar {
  width: 6px;
}
.dashboard-content::-webkit-scrollbar-track {
  background: transparent;
}
.dashboard-content::-webkit-scrollbar-thumb {
  background: rgba(0, 212, 255, 0.2);
  border-radius: 4px;
}
.dashboard-content::-webkit-scrollbar-thumb:hover {
  background: rgba(0, 212, 255, 0.4);
}

.chart-paper {
  background: #ffffff;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 24px;
  box-shadow:
    inset 0 0 0 1px rgba(0, 0, 0, 0.1),
    0 4px 15px rgba(0, 0, 0, 0.3);
  width: 100%;
  overflow: hidden;
}
.fade-in {
  animation: fadeIn 0.4s ease-in-out;
}
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(5px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}
</style>
