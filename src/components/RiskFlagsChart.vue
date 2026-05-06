<template>
  <div class="chart-container">
    <h3 class="chart-title">Overall Risk Ledger</h3>
    <div ref="chartRef" class="d3-wrapper"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

// Données attendues : { green: 15, blue: 5, yellow: 2, red: 1 }
interface RiskData {
  green: number
  blue: number
  yellow: number
  red: number
}

const props = defineProps<{ data: RiskData }>()
const chartRef = ref(null)

const drawChart = () => {
  d3.select(chartRef.value).selectAll('*').remove()

  const margin = { top: 50, right: 20, bottom: 20, left: 20 }
  const width = 600 - margin.left - margin.right
  const height = 150 - margin.top - margin.bottom // Graphique peu haut

  const svg = d3
    .select(chartRef.value)
    .append('svg')
    .attr(
      'viewBox',
      `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`,
    )
    .style('background-color', '#ffffff')
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`)

  const total = props.data.green + props.data.blue + props.data.yellow + props.data.red
  if (total === 0) return

  // Préparation des segments
  const segments = [
    { key: 'Green flag', value: props.data.green, color: '#2ecc71' },
    { key: 'Mitigated risk', value: props.data.blue, color: '#3498db' },
    { key: 'Yellow flag', value: props.data.yellow, color: '#f1c40f' },
    { key: 'Red flag', value: props.data.red, color: '#e74c3c' },
  ]

  let currentX = 0
  const x = d3.scaleLinear().domain([0, total]).range([0, width])

  // Dessin des barres
  segments.forEach((segment) => {
    const barWidth = x(segment.value)
    if (barWidth > 0) {
      svg
        .append('rect')
        .attr('x', currentX)
        .attr('y', 0)
        .attr('width', barWidth)
        .attr('height', 40)
        .attr('fill', segment.color)
        .append('title')
        .text(
          `${segment.key}: ${segment.value} flags (${Math.round((segment.value / total) * 100)}%)`,
        )

      // Texte à l'intérieur de la barre si assez large
      if (barWidth > 30) {
        svg
          .append('text')
          .attr('x', currentX + barWidth / 2)
          .attr('y', 25)
          .attr('text-anchor', 'middle')
          .style('fill', segment.key.includes('Yellow') ? '#333333' : '#ffffff') // Jaune nécessite texte foncé
          .style('font-size', '14px')
          .style('font-weight', 'bold')
          .text(segment.value)
      }
      currentX += barWidth
    }
  })

  // Légende
  const legend = svg.append('g').attr('transform', `translate(0, -35)`)
  let legendX = 0
  segments.forEach((segment) => {
    legend
      .append('rect')
      .attr('x', legendX)
      .attr('y', 0)
      .attr('width', 12)
      .attr('height', 12)
      .attr('fill', segment.color)
    legend
      .append('text')
      .attr('x', legendX + 16)
      .attr('y', 10)
      .text(segment.key)
      .style('font-size', '12px')
      .style('fill', '#333')
    legendX += 100
  })
}

onMounted(() => drawChart())
watch(
  () => props.data,
  () => drawChart(),
  { deep: true },
)
</script>

<style scoped>
.chart-container {
  padding: 1.5rem;
  background: white;
  border-radius: 8px;
  color: #333333;
}
.chart-title {
  text-align: center;
  font-family: sans-serif;
  margin-bottom: 1rem;
}
.d3-wrapper {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
}
</style>
