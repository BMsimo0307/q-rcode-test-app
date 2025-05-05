<template>
  <div class="chart-container">
    <Doughnut :data="chartData" :options="chartOptions" />
  </div>
</template>

<script setup lang="ts">
import { Doughnut } from 'vue-chartjs'
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  ArcElement,
  CategoryScale
} from 'chart.js'

ChartJS.register(Title, Tooltip, Legend, ArcElement, CategoryScale)

const props = defineProps<{
  scanned: number
  notScanned: number
}>()

const chartData = {
  labels: ['Scannés', 'Non scannés'],
  datasets: [
    {
      label: 'Participants',
      data: [props.scanned, props.notScanned],
      backgroundColor: ['#2dd36f', '#eb445a'],
      borderWidth: 1
    }
  ]
}

const chartOptions = {
  responsive: true,
  maintainAspectRatio: false,
  plugins: {
    legend: {
      position: 'top'
    }
  }
}
</script>

<style scoped>
.chart-container {
  width: 100%;
  max-width: 300px;
  height: 300px;
  margin: 0 auto;
}
</style>
    