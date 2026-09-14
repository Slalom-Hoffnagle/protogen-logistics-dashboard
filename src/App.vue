<script setup lang="ts">
import { computed, ref } from 'vue'
import { Bar, Line } from 'vue-chartjs'
import { BarElement, CategoryScale, Chart as ChartJS, Filler, Legend, LineElement, LinearScale, PointElement, Tooltip } from 'chart.js'
import type { ChartOptions } from 'chart.js'
import metrics from './data/metrics.json'

ChartJS.register(CategoryScale, LinearScale, BarElement, PointElement, LineElement, Tooltip, Legend, Filler)

type MonthMetric = (typeof metrics.months)[number]
const selectedMonth = ref(0)
const months = metrics.months as MonthMetric[]
const monthOptions = [{ title: 'All months', value: 0 }, ...months.map((month) => ({ title: month.month, value: month.month_number }))]
const filteredMonths = computed(() => selectedMonth.value === 0 ? months : months.filter((month) => month.month_number === selectedMonth.value))
const currentScope = computed(() => selectedMonth.value === 0 ? `Jan - Dec ${metrics.year}` : filteredMonths.value[0].month)
const formatCurrency = (value: number) => new Intl.NumberFormat('en-US', { style: 'currency', currency: 'USD', maximumFractionDigits: 0 }).format(value)
const formatNumber = (value: number) => new Intl.NumberFormat('en-US').format(Math.round(value))
const formatHours = (value: number) => `${Math.floor(value).toLocaleString()}h ${Math.round((value % 1) * 60)}m`
const sum = (values: number[]) => values.reduce((total, value) => total + value, 0)

const totals = computed(() => {
  const scope = filteredMonths.value
  const calls = scope.map((month) => month.customer_service_calls)
  return {
    revenue: sum(scope.map((month) => month.revenue)),
    onTime: scope.reduce((total, month) => total + month.on_time_percentage, 0) / scope.length,
    delays: sum(scope.map((month) => month.minutes_of_delay.total)),
    calls: sum(calls.map((item) => item.received)),
    positive: sum(calls.map((item) => item.positive_sentiment)),
    negative: sum(calls.map((item) => item.negative_sentiment)),
    fuelGallons: sum(scope.map((month) => month.fuel.gallons)),
    hoursDelta: sum(scope.map((month) => month.delivery_hours.delta_expected_hours)),
  }
})

const summaryCards = computed(() => [
  { label: 'Revenue', value: formatCurrency(totals.value.revenue), detail: `${formatNumber(totals.value.fuelGallons)} gal used`, icon: 'mdi-currency-usd', color: 'teal' },
  { label: 'On-time percentage', value: `${totals.value.onTime.toFixed(1)}%`, detail: `${totals.value.hoursDelta > 0 ? '+' : ''}${formatNumber(totals.value.hoursDelta)}h vs expected`, icon: 'mdi-clock-check-outline', color: 'cyan' },
  { label: 'Minutes of delay', value: formatNumber(totals.value.delays), detail: `${formatHours(totals.value.delays / 60)} accumulated`, icon: 'mdi-timer-alert-outline', color: 'amber' },
  { label: 'Customer service calls', value: formatNumber(totals.value.calls), detail: `${formatNumber(totals.value.positive)} positive · ${formatNumber(totals.value.negative)} negative`, icon: 'mdi-headset', color: 'violet' },
])

const chartLabels = computed(() => filteredMonths.value.map((month) => month.month.slice(0, 3)))
const revenueChartData = computed(() => ({ labels: chartLabels.value, datasets: [{ label: 'Revenue', data: filteredMonths.value.map((month) => month.revenue), backgroundColor: '#2dd4bf', hoverBackgroundColor: '#5eead4', borderRadius: 5, borderSkipped: false, barPercentage: 0.58 }] }))
const onTimeChartData = computed(() => ({ labels: chartLabels.value, datasets: [{ label: 'On-time %', data: filteredMonths.value.map((month) => month.on_time_percentage), borderColor: '#67e8f9', backgroundColor: 'rgba(103, 232, 249, 0.12)', pointBackgroundColor: '#67e8f9', pointBorderColor: '#0f172a', pointBorderWidth: 2, pointRadius: 4, pointHoverRadius: 7, fill: true, tension: 0.35 }] }))
const serviceChartData = computed(() => ({ labels: chartLabels.value, datasets: [
  { label: 'Positive sentiment', data: filteredMonths.value.map((month) => month.customer_service_calls.positive_sentiment), backgroundColor: '#2dd4bf', borderRadius: 4, borderSkipped: false },
  { label: 'Negative sentiment', data: filteredMonths.value.map((month) => month.customer_service_calls.negative_sentiment), backgroundColor: '#fb7185', borderRadius: 4, borderSkipped: false },
  { label: 'Resolved satisfactorily', data: filteredMonths.value.map((month) => month.customer_service_calls.resolved_satisfactorily), backgroundColor: '#818cf8', borderRadius: 4, borderSkipped: false },
] }))

const baseOptions = { responsive: true, maintainAspectRatio: false, interaction: { mode: 'index' as const, intersect: false }, scales: { x: { grid: { display: false }, ticks: { color: '#94a3b8' } }, y: { grid: { color: 'rgba(148, 163, 184, 0.12)' }, ticks: { color: '#94a3b8' } } }, plugins: { legend: { labels: { color: '#cbd5e1', usePointStyle: true, padding: 20 } } } }
const revenueChartOptions: ChartOptions<'bar'> = { ...baseOptions, plugins: { ...baseOptions.plugins, tooltip: { callbacks: { label: (context) => ` ${formatCurrency(context.parsed.y ?? 0)}` } } }, scales: { ...baseOptions.scales, y: { ...baseOptions.scales.y, ticks: { ...baseOptions.scales.y.ticks, callback: (value) => `$${Number(value) / 1000000}M` } } } }
const onTimeChartOptions: ChartOptions<'line'> = { ...baseOptions, plugins: { ...baseOptions.plugins, legend: { display: false }, tooltip: { callbacks: { label: (context) => ` ${context.parsed.y?.toFixed(1)}% on time` } } }, scales: { ...baseOptions.scales, y: { ...baseOptions.scales.y, min: 88, max: 100, ticks: { ...baseOptions.scales.y.ticks, callback: (value) => `${value}%` } } } }
const serviceChartOptions: ChartOptions<'bar'> = { ...baseOptions, scales: { ...baseOptions.scales, y: { ...baseOptions.scales.y, beginAtZero: true } }, plugins: { ...baseOptions.plugins, tooltip: { callbacks: { label: (context) => ` ${formatNumber(context.parsed.y ?? 0)} calls` } } } }
</script>

<template>
  <v-app>
  <v-navigation-drawer permanent width="248" class="sidebar">
    <div class="brand px-6 py-7"><div class="brand-mark"><v-icon icon="mdi-arrow-collapse-right" size="19" /></div><div><div class="brand-name">FastForward</div><div class="brand-subtitle">LOGISTICS</div></div></div>
    <v-list nav class="px-3 mt-3">
      <v-list-subheader class="nav-label">FastForward Logisitics</v-list-subheader>
      <v-list-item active rounded="lg" prepend-icon="mdi-view-dashboard-outline" title="Overview" />
      <v-list-item rounded="lg" prepend-icon="mdi-truck-outline" title="Fleet" />
      <v-list-item rounded="lg" prepend-icon="mdi-map-marker-path" title="Routes" />
      <v-list-item rounded="lg" prepend-icon="mdi-account-group-outline" title="Customers" />
      <v-list-subheader class="nav-label mt-6">Manage</v-list-subheader>
      <v-list-item rounded="lg" prepend-icon="mdi-file-chart-outline" title="Reports" />
      <v-list-item rounded="lg" prepend-icon="mdi-cog-outline" title="Settings" />
    </v-list>
    <div class="sidebar-footer px-5"><div class="support-card pa-4"><v-icon icon="mdi-lifebuoy" color="teal-lighten-2" class="mb-2" /><div class="text-body-2 font-weight-medium">Need a hand?</div><div class="text-caption text-medium-emphasis mt-1">Contact operations support</div><v-btn size="small" variant="text" color="teal-lighten-2" class="px-0 mt-2">Get support <v-icon end size="14">mdi-arrow-right</v-icon></v-btn></div><div class="user-row mt-5"><v-avatar color="deep-purple-lighten-2" size="34">JW</v-avatar><div class="ml-3"><div class="text-body-2">Jordan Wells</div><div class="text-caption text-medium-emphasis">Operations lead</div></div><v-icon class="ml-auto" size="18">mdi-dots-vertical</v-icon></div></div>
  </v-navigation-drawer>
  <v-app-bar flat class="topbar px-4 px-md-8"><div class="d-flex align-center"><span class="text-caption text-medium-emphasis mr-2">Workspace</span><span class="text-caption">/</span><span class="text-caption font-weight-medium ml-2">Overview</span></div><v-spacer /><v-btn icon="mdi-bell-outline" variant="text" size="small" class="mr-2" aria-label="Notifications" /><v-divider vertical class="mr-4" /><v-select v-model="selectedMonth" :items="monthOptions" item-title="title" item-value="value" density="compact" variant="outlined" hide-details class="month-select" prepend-inner-icon="mdi-calendar-month-outline" aria-label="Filter by month" /></v-app-bar>
  <v-main class="main-area"><v-container fluid class="dashboard-page px-5 px-md-8">
    <div class="d-flex flex-wrap align-end justify-space-between mb-8 gap-4"><div><div class="eyebrow">Performance snapshot <span class="live-dot" /> Live data</div><h1 class="page-title mt-2">Good morning, Jordan</h1><p class="page-subtitle mt-2">Here is what is happening across your network for <strong>{{ currentScope }}</strong>.</p></div><v-btn color="teal-accent-3" variant="flat" prepend-icon="mdi-download-outline" class="export-btn">Export report</v-btn></div>
    <v-row class="mb-2"><v-col v-for="card in summaryCards" :key="card.label" cols="12" sm="6" xl="3"><v-card class="metric-card pa-5" rounded="lg" elevation="0"><div class="d-flex justify-space-between align-start"><div class="metric-label">{{ card.label }}</div><v-icon :icon="card.icon" :color="`${card.color}-lighten-2`" size="21" /></div><div class="metric-value mt-4">{{ card.value }}</div><div class="metric-detail mt-2">{{ card.detail }}</div></v-card></v-col></v-row>
    <v-row class="mt-2"><v-col cols="12" lg="6"><v-card class="chart-card pa-5" rounded="lg" elevation="0"><div class="card-heading"><div><div class="section-kicker">Revenue</div><h2>Monthly revenue</h2></div><v-icon icon="mdi-dots-horizontal" /></div><div class="chart-legend-note">Gross revenue in USD</div><div class="chart-wrap"><Bar :data="revenueChartData" :options="revenueChartOptions" /></div></v-card></v-col><v-col cols="12" lg="6"><v-card class="chart-card pa-5" rounded="lg" elevation="0"><div class="card-heading"><div><div class="section-kicker">Reliability</div><h2>On-time percentage</h2></div><v-icon icon="mdi-dots-horizontal" /></div><div class="chart-legend-note">Share of deliveries arriving on schedule</div><div class="chart-wrap"><Line :data="onTimeChartData" :options="onTimeChartOptions" /></div></v-card></v-col></v-row>
    <v-row class="mt-2"><v-col cols="12"><v-card class="chart-card pa-5" rounded="lg" elevation="0"><div class="card-heading"><div><div class="section-kicker">Customer experience</div><h2>Customer service calls</h2></div><v-icon icon="mdi-dots-horizontal" /></div><div class="chart-legend-note">Sentiment and resolution outcomes by month</div><div class="chart-wrap service-chart"><Bar :data="serviceChartData" :options="serviceChartOptions" /></div></v-card></v-col></v-row>
    <div class="footer-note mt-6"><span>Last refreshed today at 08:42 AM</span><span class="status-badge"><span class="live-dot" /> All systems operational</span></div>
  </v-container></v-main>
  </v-app>
</template>
