<template>
  <div class="chart-container">
    <h3 class="chart-title">Climate Impact: Projected Emissions (tCO2e)</h3>
    <div ref="areaChartRef" class="d3-wrapper"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

interface CarbonImpactDatum {
  year: number
  bau: number
  project: number
}

const props = defineProps<{
  data: CarbonImpactDatum[]
}>()

const areaChartRef = ref(null)

const drawChart = () => {
  d3.select(areaChartRef.value).selectAll('*').remove()

  const margin = { top: 40, right: 30, bottom: 40, left: 60 } // Top augmenté pour la légende
  const width = 600 - margin.left - margin.right
  const height = 300 - margin.top - margin.bottom

  const svg = d3
    .select(areaChartRef.value)
    .append('svg')
    .attr(
      'viewBox',
      `0 0 ${width + margin.left + margin.right} ${height + margin.top + margin.bottom}`,
    )
    .style('background-color', '#ffffff')
    .append('g')
    .attr('transform', `translate(${margin.left},${margin.top})`)

  const x = d3
    .scaleLinear()
    .domain(props.data.length ? (d3.extent(props.data, (d) => d.year) as [number, number]) : [0, 1])
    .range([0, width])

  const yMax = d3.max(props.data, (d) => Math.max(d.bau, d.project) ?? 0) ?? 0
  const y = d3
    .scaleLinear()
    .domain([0, yMax * 1.1])
    .range([height, 0])

  // AXES (Isolation CSS)
  const xAxis = svg
    .append('g')
    .attr('transform', `translate(0,${height})`)
    .call(d3.axisBottom(x).tickFormat(d3.format('d')))
  xAxis.selectAll('text').style('fill', '#333333')
  xAxis.selectAll('path, line').style('stroke', '#cccccc')

  const yAxis = svg.append('g').call(d3.axisLeft(y))
  yAxis.selectAll('text').style('fill', '#333333')
  yAxis.selectAll('path, line').style('stroke', '#cccccc')

  // AIRE VERTE
  const areaGenerator = d3
    .area<CarbonImpactDatum>()
    .x((d) => x(d.year) as number)
    .y0((d) => y(d.project) as number)
    .y1((d) => y(d.bau) as number)

  svg
    .append('path')
    .datum(props.data)
    .attr('fill', '#2ecc71')
    .attr('fill-opacity', 0.3)
    .attr('d', areaGenerator)

  // LIGNE BAU (Rouge pointillée)
  const lineBAU = d3
    .line<CarbonImpactDatum>()
    .x((d) => x(d.year))
    .y((d) => y(d.bau))
  svg
    .append('path')
    .datum(props.data)
    .attr('fill', 'none')
    .attr('stroke', '#e74c3c')
    .attr('stroke-width', 2)
    .attr('stroke-dasharray', '5,5')
    .attr('d', lineBAU)

  // LIGNE PROJET (Verte pleine)
  const lineProject = d3
    .line<CarbonImpactDatum>()
    .x((d) => x(d.year) as number)
    .y((d) => y(d.project) as number)
  svg
    .append('path')
    .datum(props.data)
    .attr('fill', 'none')
    .attr('stroke', '#27ae60')
    .attr('stroke-width', 3)
    .attr('d', lineProject)

  // --- LÉGENDE ---
  const legend = svg.append('g').attr('transform', `translate(0, -30)`)

  // Légende BAU
  legend
    .append('line')
    .attr('x1', 0)
    .attr('x2', 20)
    .attr('y1', 5)
    .attr('y2', 5)
    .attr('stroke', '#e74c3c')
    .attr('stroke-width', 2)
    .attr('stroke-dasharray', '3,3')
  legend
    .append('text')
    .attr('x', 25)
    .attr('y', 9)
    .text('BAU Scenario (Baseline)')
    .style('font-size', '12px')
    .style('fill', '#333333')

  // Légende Projet
  legend
    .append('line')
    .attr('x1', 180)
    .attr('x2', 200)
    .attr('y1', 5)
    .attr('y2', 5)
    .attr('stroke', '#27ae60')
    .attr('stroke-width', 3)
  legend
    .append('text')
    .attr('x', 205)
    .attr('y', 9)
    .text('Project Scenario')
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
