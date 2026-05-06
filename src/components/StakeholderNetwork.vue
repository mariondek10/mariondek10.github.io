<template>
  <div class="chart-container">
    <h3 class="chart-title">Project Stakeholders</h3>
    <div ref="networkRef" class="d3-wrapper"></div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted, watch } from 'vue'
import * as d3 from 'd3'

// Structure : nodes (nœuds) et links (liens)
interface Node extends d3.SimulationNodeDatum {
  id: string
  group: number
}
interface Link extends d3.SimulationLinkDatum<Node> {
  source: string | Node
  target: string | Node
}
interface NetworkData {
  nodes: Node[]
  links: Link[]
}

const props = defineProps<{ data: NetworkData }>()
const networkRef = ref(null)

const drawChart = () => {
  d3.select(networkRef.value).selectAll('*').remove()

  const width = 600
  const height = 400

  const svg = d3
    .select(networkRef.value)
    .append('svg')
    .attr('viewBox', `0 0 ${width} ${height}`)
    .style('background-color', '#ffffff')

  // 🚨 LA CORRECTION EST ICI :
  // On clone les données pour que D3 puisse injecter ses coordonnées (x,y)
  // sans être bloqué par la protection "Read-Only" des props de Vue 3.
  const clonedNodes = props.data.nodes.map((d) => ({ ...d }))
  const clonedLinks = props.data.links.map((d) => ({ ...d }))

  // Couleurs selon les groupes (1: Projet, 2: Stakeholder, 3: SDG)
  const color = d3
    .scaleOrdinal<number, string>()
    .domain([1, 2, 3])
    .range(['#2c3e50', '#3498db', '#e67e22'])

  const simulation = d3
    .forceSimulation(clonedNodes)
    .force(
      'link',
      d3
        .forceLink<Node, Link>(clonedLinks)
        .id((d) => d.id)
        .distance(80),
    )
    .force('charge', d3.forceManyBody().strength(-300))
    .force('center', d3.forceCenter(width / 2, height / 2))

  const link = svg
    .append('g')
    .attr('stroke', '#cccccc')
    .attr('stroke-opacity', 0.6)
    .selectAll('line')
    .data(clonedLinks)
    .enter()
    .append('line')
    .attr('stroke-width', 2)

  const node = svg
    .append('g')
    .selectAll('circle')
    .data(clonedNodes)
    .enter()
    .append('circle')
    .attr('r', (d) => (d.group === 1 ? 30 : 20))
    .attr('fill', (d) => color(d.group))
    .call(
      d3
        .drag<SVGCircleElement, Node>()
        .on('start', (event, d) => {
          if (!event.active) simulation.alphaTarget(0.3).restart()
          d.fx = d.x
          d.fy = d.y
        })
        .on('drag', (event, d) => {
          d.fx = event.x
          d.fy = event.y
        })
        .on('end', (event, d) => {
          if (!event.active) simulation.alphaTarget(0)
          d.fx = null
          d.fy = null
        }),
    )

  const labels = svg
    .append('g')
    .selectAll('text')
    .data(clonedNodes)
    .enter()
    .append('text')
    .text((d) => d.id)
    .attr('font-size', '15px')
    .attr('fill', '#333333')
    .attr('dx', 30)
    .attr('dy', 30)

  simulation.on('tick', () => {
    link
      .attr('x1', (d: Link) => (d.source as Node).x ?? 0)
      .attr('y1', (d: Link) => (d.source as Node).y ?? 0)
      .attr('x2', (d: Link) => (d.target as Node).x ?? 0)
      .attr('y2', (d: Link) => (d.target as Node).y ?? 0)

    node.attr('cx', (d: Node) => d.x ?? 0).attr('cy', (d: Node) => d.y ?? 0)

    labels.attr('x', (d: Node) => d.x ?? 0).attr('y', (d: Node) => d.y ?? 0)
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
  border: 1px solid #eee;
}
</style>
