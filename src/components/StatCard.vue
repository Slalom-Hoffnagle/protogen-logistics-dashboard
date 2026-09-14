<script setup lang="ts">
defineProps<{
  title: string
  value: string
  change: string
  icon: string
  positive?: boolean
}>()
</script>

<template>
  <v-card rounded="xl" elevation="0" class="pa-4 h-100 stat-card">
    <div class="d-flex align-center justify-space-between mb-4">
      <div>
        <p class="text-caption text-medium-emphasis mb-1">{{ title }}</p>
        <div class="text-h5 font-weight-bold">{{ value }}</div>
      </div>

      <v-avatar color="primary" variant="tonal" size="48">
        <v-icon :icon="icon" />
      </v-avatar>
    </div>

    <v-chip :color="positive ? 'success' : 'warning'" variant="tonal" size="small">
      <v-icon start size="small">{{ positive ? 'mdi-arrow-up-right' : 'mdi-arrow-down-right' }}</v-icon>
      {{ change }}
    </v-chip>
  </v-card>
</template>
