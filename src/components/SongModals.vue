<script setup>
defineProps({
  song: {
    type: Object,
    default: null,
  },
  showHint: Boolean,
  showAnswer: Boolean,
});

const emit = defineEmits(['close-hint', 'close-answer']);
</script>

<template>
  <Transition name="modal">
    <div v-if="showHint" class="game-modal" @click.self="emit('close-hint')">
      <div class="game-modal-card hint-modal-card">
        <button type="button" class="modal-close" @click="emit('close-hint')">×</button>
        <div class="modal-icon">💡</div>
        <div class="modal-label">ПОДСКАЗКА</div>
        <div class="modal-main-text">{{ song?.hint }}</div>
        <button type="button" class="modal-action" @click="emit('close-hint')">Понятно</button>
      </div>
    </div>
  </Transition>

  <Transition name="modal">
    <div v-if="showAnswer" class="game-modal" @click.self="emit('close-answer')">
      <div class="game-modal-card answer-modal-card">
        <button type="button" class="modal-close" @click="emit('close-answer')">×</button>
        <div class="modal-icon">🎵</div>
        <div class="modal-label">ПРАВИЛЬНЫЙ ОТВЕТ</div>
        <div class="answer-artist">{{ song?.artist }}</div>
        <div class="answer-title">{{ song?.title }}</div>
        <button type="button" class="modal-action" @click="emit('close-answer')">Закрыть</button>
      </div>
    </div>
  </Transition>
</template>
