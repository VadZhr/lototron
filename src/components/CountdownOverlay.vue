<script setup>
defineProps({
  active: Boolean,
  value: Number,
});
</script>

<template>
  <Transition name="countdown">
    <div v-if="active" class="countdown-overlay">
      <div :key="value" class="countdown-number">
        {{ value }}
      </div>
      <div class="countdown-label">УГАДАЙ ПЕСНЮ</div>
    </div>
  </Transition>
</template>
