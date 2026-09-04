<script setup>
defineProps({
  pulled: {
    type: Array,
    required: true,
  },
  total: {
    type: Number,
    required: true,
  },
});
</script>

<template>
  <div class="history">
    <div class="history-header">
      <h2>Выпавшие числа</h2>
      <span>{{ pulled.length }} / {{ total }}</span>
    </div>

    <div v-if="pulled.length" class="numbers">
      <div
        v-for="(value, index) in pulled"
        :key="value"
        class="history-number"
        :class="{ latest: index === pulled.length - 1 }"
      >
        {{ value }}
      </div>
    </div>

    <div v-else class="empty-history">
      Здесь появятся выпавшие числа
    </div>
  </div>
</template>
