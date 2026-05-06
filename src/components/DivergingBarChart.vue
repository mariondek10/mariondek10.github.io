<template>
  <div class="chart-container">
    <h3 class="chart-title">Positive vs Negative Findings</h3>
    <div ref="chartRef" class="d3-wrapper"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

interface DataItem {
  category: string
  positive: number
  negative: number
}

const props = defineProps({
  data: {
    type: Array as () => DataItem[],
    required: true,
  },
})

const chartRef = ref(null)

const drawChart = () => {
  d3.select(chartRef.value).selectAll('*').remove()

  const margin = { top: 40, right: 30, bottom: 40, left: 100 } // Augmentation du top pour la légende
  const width = 600 - margin.left - margin.right
  const height = 400 - margin.top - margin.bottom

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

  const maxVal = d3.max(props.data, (d) => Math.max(d.positive, Math.abs(d.negative))) ?? 0
  const x = d3.scaleLinear().domain([-maxVal, maxVal]).range([0, width])
  const y = d3
    .scaleBand()
    .domain(props.data.map((d) => d.category))
    .range([0, height])
    .padding(0.2)

  // AXES (Avec couleurs forcées pour contrer le dark mode)
  const xAxis = svg.append('g').attr('transform', `translate(0,${height})`).call(d3.axisBottom(x))
  xAxis.selectAll('text').style('fill', '#333333').style('font-size', '11px') // Forcer texte foncé
  xAxis.selectAll('path, line').style('stroke', '#cccccc') // Forcer lignes grises

  const yAxis = svg.append('g').call(d3.axisLeft(y).tickSize(0))
  yAxis
    .selectAll('text')
    .style('fill', '#333333')
    .style('font-size', '12px')
    .style('font-weight', 'bold')
  yAxis.select('.domain').remove() // Enlever la ligne verticale de l'axe Y pour faire plus propre

  // Ligne centrale (Le Zéro)
  svg
    .append('line')
    .attr('x1', x(0))
    .attr('x2', x(0))
    .attr('y1', 0)
    .attr('y2', height)
    .attr('stroke', '#333333')
    .attr('stroke-width', 1)

  // BARRES POSITIVES
  svg
    .selectAll('.bar-positive')
    .data(props.data)
    .enter()
    .append('rect')
    .attr('class', 'bar-positive')
    .attr('x', x(0))
    .attr('y', (d) => y(d.category) ?? 0)
    .attr('width', (d) => x(d.positive) - x(0))
    .attr('height', y.bandwidth())
    .attr('fill', '#2ecc71')
    .append('title') // Infobulle simple
    .text((d) => `${d.category} Positive: ${d.positive}`)

  // BARRES NÉGATIVES
  svg
    .selectAll('.bar-negative')
    .data(props.data)
    .enter()
    .append('rect')
    .attr('class', 'bar-negative')
    .attr('x', (d) => x(d.negative))
    .attr('y', (d) => y(d.category) ?? 0)
    .attr('width', (d) => x(0) - x(d.negative))
    .attr('height', y.bandwidth())
    .attr('fill', '#e74c3c')
    .append('title')
    .text((d) => `${d.category} Negative: ${Math.abs(d.negative)}`)

  // --- LÉGENDE ---
  const legend = svg.append('g').attr('transform', `translate(0, -30)`)

  // Positif
  legend
    .append('rect')
    .attr('x', width / 2 + 20)
    .attr('y', 0)
    .attr('width', 15)
    .attr('height', 15)
    .attr('fill', '#2ecc71')
  legend
    .append('text')
    .attr('x', width / 2 + 40)
    .attr('y', 12)
    .text('Positive Findings')
    .style('font-size', '12px')
    .style('fill', '#333333')

  // Négatif
  legend
    .append('rect')
    .attr('x', 20)
    .attr('y', 0)
    .attr('width', 15)
    .attr('height', 15)
    .attr('fill', '#e74c3c')
  legend
    .append('text')
    .attr('x', 40)
    .attr('y', 12)
    .text('Negative Findings')
    .style('font-size', '12px')
    .style('fill', '#333333')
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
  /* J'ai forcé la couleur du titre pour qu'il ne soit pas blanc sur blanc */
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
