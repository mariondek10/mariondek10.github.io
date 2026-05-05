<template>
  <div class="chart-container">
    <h3 class="chart-title">Pillars Health Score (%)</h3>
    <div ref="radarChartRef" class="d3-wrapper"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

// Données attendues (pourcentage de positif) : [{ axis: 'Design', value: 90 }, ...]
interface RadarDatum {
  axis: string
  value: number
}
const props = defineProps<{ data: RadarDatum[] }>()
const radarChartRef = ref(null)

const drawChart = () => {
  d3.select(radarChartRef.value).selectAll('*').remove()

  const margin = { top: 50, right: 50, bottom: 50, left: 50 }
  const width = 400
  const height = 400
  const radius = Math.min(width, height) / 2

  const svg = d3
    .select(radarChartRef.value)
    .append('svg')
    .attr(
      'viewBox',
      `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`,
    )
    .style('background-color', '#ffffff')
    .append('g')
    .attr('transform', `translate(${width / 2 + margin.left},${height / 2 + margin.top})`)

  const angleSlice = (Math.PI * 2) / props.data.length
  const rScale = d3.scaleLinear().range([0, radius]).domain([0, 100])

  // Dessin de la grille de fond (cercles)
  const ticks = [20, 40, 60, 80, 100]
  ticks.forEach((t) => {
    svg
      .append('circle')
      .attr('r', rScale(t))
      .style('fill', 'none')
      .style('stroke', '#e0e0e0')
      .style('stroke-dasharray', '3,3')
    svg
      .append('text')
      .attr('y', -rScale(t) - 4)
      .style('fill', '#888')
      .style('font-size', '10px')
      .text(t)
  })

  // Axes (lignes depuis le centre)
  const axis = svg.selectAll('.axis').data(props.data).enter().append('g').attr('class', 'axis')
  axis
    .append('line')
    .attr('x1', 0)
    .attr('y1', 0)
    .attr('x2', (d, i) => rScale(100) * Math.cos(angleSlice * i - Math.PI / 2))
    .attr('y2', (d, i) => rScale(100) * Math.sin(angleSlice * i - Math.PI / 2))
    .style('stroke', '#e0e0e0')
    .style('stroke-width', '1px')

  // Labels des axes
  axis
    .append('text')
    .attr('x', (d, i) => rScale(115) * Math.cos(angleSlice * i - Math.PI / 2))
    .attr('y', (d, i) => rScale(115) * Math.sin(angleSlice * i - Math.PI / 2))
    .text((d) => d.axis)
    .style('text-anchor', 'middle')
    .style('fill', '#333333')
    .style('font-size', '12px')
    .style('font-weight', 'bold')

  // Générateur de la forme
  const radarLine = d3
    .lineRadial<RadarDatum>()
    .angle((d, i) => i * angleSlice)
    .radius((d) => rScale(d.value))
    .curve(d3.curveLinearClosed)

  // Dessin de l'aire colorée
  svg
    .append('path')
    .datum(props.data)
    .attr('d', radarLine)
    .style('fill', '#27ae60')
    .style('fill-opacity', 0.4)
    .style('stroke', '#27ae60')
    .style('stroke-width', 2)
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
  max-width: 500px;
  margin: 0 auto;
}
</style>
