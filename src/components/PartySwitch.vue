<script setup>
defineProps({
  partyType: {
    type: String,
    required: true,
  },
  disabled: Boolean,
});

const emit = defineEmits(['change']);
</script>

<template>
  <div class="party-switch">
    <button
      type="button"
      class="party-switch-button"
      :class="{ active: partyType === 'girls' }"
      :disabled="disabled"
      @click="emit('change', 'girls')"
    >
      <span class="switch-icon">♡</span>
      Девичник
    </button>

    <button
      type="button"
      class="party-switch-button"
      :class="{ active: partyType === 'boys' }"
      :disabled="disabled"
      @click="emit('change', 'boys')"
    >
      <span class="switch-icon">♠</span>
      Мальчишник
    </button>
  </div>
</template>
