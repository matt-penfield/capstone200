<template>
  <v-app>
    <v-app-bar elevation="2" color="surface">
      <v-app-bar-title>
        <span class="text-h6 font-weight-bold">Fast Forward Logistics</span>
      </v-app-bar-title>
      <template v-slot:append>
        <v-select
          v-model="selectedMonth"
          :items="monthOptions"
          item-title="label"
          item-value="value"
          variant="outlined"
          density="compact"
          hide-details
          style="max-width: 160px"
        />
      </template>
    </v-app-bar>

    <v-main>
      <v-container fluid class="pa-6">
        <!-- Summary Cards -->
        <v-row>
          <v-col cols="12" sm="6" lg="3" v-for="card in summaryCards" :key="card.title">
            <v-card class="pa-4" variant="elevated" elevation="1">
              <div class="text-caption text-medium-emphasis text-uppercase tracking-wide">
                {{ card.title }}
              </div>
              <div class="d-flex align-center mt-2">
                <span class="text-h4 font-weight-bold">{{ card.value }}</span>
                <v-chip
                  v-if="card.change !== null"
                  :color="card.changeColor"
                  size="small"
                  variant="tonal"
                  class="ml-3"
                >
                  <v-icon size="14" :icon="card.changeIcon" />
                  {{ card.changeText }}
                </v-chip>
              </div>
            </v-card>
          </v-col>
        </v-row>

        <!-- Charts Row 1: Bar + Line -->
        <v-row class="mt-4">
          <v-col cols="12" md="6">
            <v-card class="pa-4" variant="elevated" elevation="1">
              <div class="text-subtitle-1 font-weight-medium mb-4">Monthly Shipment Volume</div>
              <Bar :data="barChartData" :options="barChartOptions" />
            </v-card>
          </v-col>
          <v-col cols="12" md="6">
            <v-card class="pa-4" variant="elevated" elevation="1">
              <div class="text-subtitle-1 font-weight-medium mb-4">On-Time Delivery Rate</div>
              <Line :data="lineChartData" :options="lineChartOptions" />
            </v-card>
          </v-col>
        </v-row>

        <!-- Chart Row 2: Full-width Area -->
        <v-row class="mt-4">
          <v-col cols="12">
            <v-card class="pa-4" variant="elevated" elevation="1">
              <div class="text-subtitle-1 font-weight-medium mb-4">Open Exceptions Trend</div>
              <Line :data="areaChartData" :options="areaChartOptions" />
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'
import { Bar, Line } from 'vue-chartjs'
import {
  Chart as ChartJS,
  CategoryScale,
  LinearScale,
  BarElement,
  PointElement,
  LineElement,
  Filler,
  Title,
  Tooltip,
  Legend,
} from 'chart.js'
import metricsData from './data/metrics.json'

ChartJS.register(
  CategoryScale,
  LinearScale,
  BarElement,
  PointElement,
  LineElement,
  Filler,
  Title,
  Tooltip,
  Legend
)

interface MonthData {
  month: string
  year: number
  shipmentVolume: number
  onTimeDeliveryRate: number
  regionalPerformance: number
  openExceptions: number
}

const data: MonthData[] = metricsData as MonthData[]

const monthOptions = [
  { label: 'All Months', value: 'all' },
  ...data.map((d) => ({ label: d.month, value: d.month })),
]

const selectedMonth = ref<string>('all')

const filteredData = computed(() => {
  if (selectedMonth.value === 'all') return data
  return data.filter((d) => d.month === selectedMonth.value)
})

const selectedIndex = computed(() => {
  if (selectedMonth.value === 'all') return -1
  return data.findIndex((d) => d.month === selectedMonth.value)
})

// Summary cards
const summaryCards = computed(() => {
  const fd = filteredData.value
  const isAll = selectedMonth.value === 'all'

  const totalVolume = fd.reduce((sum, d) => sum + d.shipmentVolume, 0)
  const avgOnTime = fd.reduce((sum, d) => sum + d.onTimeDeliveryRate, 0) / fd.length
  const avgRegional = fd.reduce((sum, d) => sum + d.regionalPerformance, 0) / fd.length
  const totalExceptions = fd.reduce((sum, d) => sum + d.openExceptions, 0)

  const idx = selectedIndex.value
  const prev = idx > 0 ? data[idx - 1] : null

  function getChange(current: number, previous: number | null) {
    if (previous === null || isAll) return { change: null, changeColor: '', changeIcon: '', changeText: '' }
    const diff = ((current - previous) / previous) * 100
    return {
      change: diff,
      changeColor: diff >= 0 ? 'success' : 'error',
      changeIcon: diff >= 0 ? 'mdi-arrow-up' : 'mdi-arrow-down',
      changeText: `${Math.abs(diff).toFixed(1)}%`,
    }
  }

  const volumeChange = getChange(
    isAll ? totalVolume : fd[0]?.shipmentVolume ?? 0,
    prev?.shipmentVolume ?? null
  )
  const onTimeChange = getChange(
    isAll ? avgOnTime : fd[0]?.onTimeDeliveryRate ?? 0,
    prev?.onTimeDeliveryRate ?? null
  )
  const regionalChange = getChange(
    isAll ? avgRegional : fd[0]?.regionalPerformance ?? 0,
    prev?.regionalPerformance ?? null
  )
  const exceptionsChange = getChange(
    isAll ? totalExceptions : fd[0]?.openExceptions ?? 0,
    prev?.openExceptions ?? null
  )

  return [
    {
      title: 'Shipment Volume',
      value: isAll ? totalVolume.toLocaleString() : fd[0]?.shipmentVolume.toLocaleString() ?? '—',
      ...volumeChange,
    },
    {
      title: 'On-Time Delivery',
      value: `${(isAll ? avgOnTime : fd[0]?.onTimeDeliveryRate ?? 0).toFixed(1)}%`,
      ...onTimeChange,
    },
    {
      title: 'Regional Performance',
      value: `${(isAll ? avgRegional : fd[0]?.regionalPerformance ?? 0).toFixed(1)}%`,
      ...regionalChange,
    },
    {
      title: 'Open Exceptions',
      value: isAll ? totalExceptions.toLocaleString() : (fd[0]?.openExceptions ?? 0).toLocaleString(),
      ...exceptionsChange,
    },
  ]
})

// Color palette
const colors = {
  primary: '#6366f1',
  primaryLight: 'rgba(99, 102, 241, 0.2)',
  secondary: '#06b6d4',
  secondaryLight: 'rgba(6, 182, 212, 0.2)',
  accent: '#f59e0b',
  accentLight: 'rgba(245, 158, 11, 0.15)',
  danger: '#ef4444',
  dangerLight: 'rgba(239, 68, 68, 0.15)',
}

const chartTextColor = 'rgba(0, 0, 0, 0.6)'
const chartGridColor = 'rgba(0, 0, 0, 0.08)'

// Bar chart - Shipment Volume
const barChartData = computed(() => {
  const labels = data.map((d) => d.month)
  const values = data.map((d) => d.shipmentVolume)
  const bgColors = data.map((_, i) =>
    selectedMonth.value === 'all' || data[i].month === selectedMonth.value
      ? colors.primary
      : 'rgba(99, 102, 241, 0.2)'
  )

  return {
    labels,
    datasets: [
      {
        label: 'Shipment Volume',
        data: values,
        backgroundColor: bgColors,
        borderRadius: 4,
      },
    ],
  }
})

const barChartOptions = {
  responsive: true,
  maintainAspectRatio: true,
  plugins: {
    legend: { display: false },
  },
  scales: {
    x: { ticks: { color: chartTextColor }, grid: { display: false } },
    y: { ticks: { color: chartTextColor }, grid: { color: chartGridColor } },
  },
}

// Line chart - On-Time Delivery Rate
const lineChartData = computed(() => {
  const labels = data.map((d) => d.month)
  const values = data.map((d) => d.onTimeDeliveryRate)

  return {
    labels,
    datasets: [
      {
        label: 'On-Time %',
        data: values,
        borderColor: colors.secondary,
        backgroundColor: colors.secondaryLight,
        tension: 0.3,
        pointRadius: data.map((d) =>
          selectedMonth.value === 'all' ? 3 : d.month === selectedMonth.value ? 7 : 3
        ),
        pointBackgroundColor: data.map((d) =>
          selectedMonth.value !== 'all' && d.month === selectedMonth.value
            ? colors.accent
            : colors.secondary
        ),
      },
    ],
  }
})

const lineChartOptions = {
  responsive: true,
  maintainAspectRatio: true,
  plugins: {
    legend: { display: false },
  },
  scales: {
    x: { ticks: { color: chartTextColor }, grid: { display: false } },
    y: {
      ticks: { color: chartTextColor, callback: (v: any) => `${v}%` },
      grid: { color: chartGridColor },
      min: 80,
      max: 100,
    },
  },
}

// Area chart - Open Exceptions
const areaChartData = computed(() => {
  const labels = data.map((d) => d.month)
  const values = data.map((d) => d.openExceptions)

  return {
    labels,
    datasets: [
      {
        label: 'Open Exceptions',
        data: values,
        borderColor: colors.danger,
        backgroundColor: colors.dangerLight,
        fill: true,
        tension: 0.4,
        pointRadius: data.map((d) =>
          selectedMonth.value === 'all' ? 3 : d.month === selectedMonth.value ? 7 : 3
        ),
        pointBackgroundColor: data.map((d) =>
          selectedMonth.value !== 'all' && d.month === selectedMonth.value
            ? colors.accent
            : colors.danger
        ),
      },
    ],
  }
})

const areaChartOptions = {
  responsive: true,
  maintainAspectRatio: true,
  plugins: {
    legend: { display: false },
  },
  scales: {
    x: { ticks: { color: chartTextColor }, grid: { display: false } },
    y: { ticks: { color: chartTextColor }, grid: { color: chartGridColor } },
  },
}
</script>

<style>
html, body {
  overflow-y: auto;
}
</style>
