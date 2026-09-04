<script setup>
const props = defineProps({
  trackNumber: { type: Number, required: true },
  isPlaying: Boolean,
  isLoading: Boolean,
  hasError: Boolean,
  isFinished: Boolean,
  currentTime: { type: Number, default: 0 },
  duration: { type: Number, default: 0 },
  progress: { type: Number, default: 0 },
  volume: { type: Number, default: 0.8 },
});

const emit = defineEmits([
  'toggle-pause',
  'replay',
  'show-hint',
  'show-answer',
  'update:volume',
]);

const formatTime = (seconds) => {
  if (!Number.isFinite(seconds)) return '0:00';
  const minutes = Math.floor(seconds / 60);
  const secs = Math.floor(seconds % 60).toString().padStart(2, '0');
  return `${minutes}:${secs}`;
};

const updateVolume = (event) => {
  emit('update:volume', Number(event.target.value));
};
</script>

<template>
  <div class="music-player">
    <div class="music-header">
      <div class="playing-status">
        <div v-if="isLoading" class="audio-loader" aria-label="Загрузка аудио"></div>
        <div v-else class="equalizer" :class="{ active: isPlaying }">
          <span></span><span></span><span></span><span></span><span></span>
        </div>

        <div class="music-status-text">
          <strong>
            {{
              isLoading
                ? 'Загрузка аудио…'
                : hasError
                  ? 'Не удалось загрузить аудио'
                  : isFinished
                    ? 'Фрагмент закончился'
                    : isPlaying
                      ? 'Сейчас играет'
                      : 'На паузе'
            }}
          </strong>
          <span>Трек №{{ trackNumber }}</span>
        </div>
      </div>

      <div class="music-time">
        {{ formatTime(currentTime) }} / {{ formatTime(duration) }}
      </div>
    </div>

    <div class="progress-track">
      <div class="progress-value" :style="{ width: `${progress}%` }"></div>
    </div>

    <div class="music-controls">
      <button
        type="button"
        class="music-control primary-control"
        :disabled="isFinished || isLoading"
        @click="emit('toggle-pause')"
      >
        {{ isPlaying ? '❚❚ Пауза' : '▶ Продолжить' }}
      </button>

      <button type="button" class="music-control" :disabled="isLoading" @click="emit('replay')">
        ↻ Заново
      </button>

      <div class="volume-control">
        <span>🔊</span>
        <input
          :value="props.volume"
          type="range"
          min="0"
          max="1"
          step="0.05"
          @input="updateVolume"
        >
      </div>
    </div>

    <div class="game-help">
      <div class="game-help-buttons">
        <button type="button" class="hint-button" @click="emit('show-hint')">💡 Подсказка</button>
        <button type="button" class="answer-button" @click="emit('show-answer')">👀 Показать ответ</button>
      </div>
    </div>
  </div>
</template>
