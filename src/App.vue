<script setup>
import {
  computed,
  onBeforeUnmount,
  onMounted,
  ref,
  watch,
} from 'vue';
import { songs } from './constants';
import LoginForm from './components/LoginForm.vue';

const isAuthorized = ref(false);

onMounted(() => {
  loadFromLocalStorage();

  isAuthorized.value =
    sessionStorage.getItem('muzloto-authorized') === 'true';
});

const handleLoginSuccess = () => {
  isAuthorized.value = true;
};

const STORAGE_KEY = 'random-number-generator';

const partyType = ref('girls');

const countdown = ref(null);
const isCountingDown = ref(false);

const showHintModal = ref(false);
const showAnswerModal = ref(false);

const startCountdown = async (number) => {
  isCountingDown.value = true;

  for (let value = 3; value >= 1; value--) {
    countdown.value = value;

    await new Promise((resolve) => {
      setTimeout(resolve, 1000);
    });
  }

  countdown.value = null;
  isCountingDown.value = false;

  await playSong(number);
};

const audio = ref(null);

const isPlaying = ref(false);
const isPaused = ref(false);
const audioFinished = ref(true);

const duration = ref(0);
const currentTime = ref(0);

const volume = ref(0.8);

const showHint = ref(false);
const showAnswer = ref(false);

const currentSong = computed(() => {
  if (lastPulled.value === null) {
    return null;
  }

  return (
    songs[partyType.value]?.[lastPulled.value] ??
    null
  );
});

const canGenerate = computed(() => {
  return (
    !isFinished.value &&
    audioFinished.value &&
    !isCountingDown.value
  );
});

const playSong = async (number) => {
  destroyAudio();

  showHint.value = false;
  showAnswer.value = false;

  duration.value = 0;
  currentTime.value = 0;

  audioFinished.value = false;
  isPaused.value = false;

  const newAudio = new Audio(
    `/audio/${partyType.value}/${number}.mp3`
  );

  newAudio.volume = volume.value;

  newAudio.addEventListener(
    'loadedmetadata',
    handleLoadedMetadata
  );

  newAudio.addEventListener(
    'timeupdate',
    handleTimeUpdate
  );

  newAudio.addEventListener(
    'ended',
    handleEnded
  );

  audio.value = newAudio;

  try {
    await newAudio.play();

    isPlaying.value = true;
  } catch (error) {
    console.error(
      'Не удалось запустить аудио',
      error
    );

    isPlaying.value = false;
    audioFinished.value = true;
  }
};

const diapason = ref({
  min: 1,
  max: 42,
});


const handleLoadedMetadata = () => {
  if (!audio.value) return;

  duration.value = audio.value.duration;
};

const handleTimeUpdate = () => {
  if (!audio.value) return;

  currentTime.value = audio.value.currentTime;
};

const handleEnded = () => {
  isPlaying.value = false;
  isPaused.value = false;
  audioFinished.value = true;

  currentTime.value = duration.value;
};

const togglePause = async () => {
  if (!audio.value || audioFinished.value) {
    return;
  }

  if (audio.value.paused) {
    try {
      await audio.value.play();

      isPlaying.value = true;
      isPaused.value = false;
    } catch (error) {
      console.error(error);
    }

    return;
  }

  audio.value.pause();

  isPlaying.value = false;
  isPaused.value = true;
};

const replaySong = async () => {
  if (!audio.value) return;

  audio.value.currentTime = 0;

  currentTime.value = 0;
  audioFinished.value = false;

  try {
    await audio.value.play();

    isPlaying.value = true;
    isPaused.value = false;
  } catch (error) {
    console.error(error);
  }
};

const changeVolume = () => {
  if (!audio.value) return;

  audio.value.volume = volume.value;
};

const progress = computed(() => {
  if (!duration.value) {
    return 0;
  }

  return (
    currentTime.value /
    duration.value
  ) * 100;
});

const formatTime = (seconds) => {
  if (!Number.isFinite(seconds)) {
    return '0:00';
  }

  const minutes =
    Math.floor(seconds / 60);

  const secs =
    Math.floor(seconds % 60)
      .toString()
      .padStart(2, '0');

  return `${minutes}:${secs}`;
};

const destroyAudio = () => {
  if (!audio.value) return;

  audio.value.pause();

  audio.value.removeEventListener(
    'loadedmetadata',
    handleLoadedMetadata
  );

  audio.value.removeEventListener(
    'timeupdate',
    handleTimeUpdate
  );

  audio.value.removeEventListener(
    'ended',
    handleEnded
  );

  audio.value = null;

  isPlaying.value = false;
};

const pulled = ref([]);
const lastPulled = ref(null);

const totalNumbers = computed(() => {
  return diapason.value.max - diapason.value.min + 1;
});

const isFinished = computed(() => {
  return pulled.value.length >= totalNumbers.value;
});

const partyLabel = computed(() => {
  return partyType.value === 'girls'
    ? 'ДЕВИЧНИК'
    : 'МАЛЬЧИШНИК';
});

const setPartyType = (type) => {
  partyType.value = type;
};

const saveToLocalStorage = () => {
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify({
      partyType: partyType.value,
      diapason: diapason.value,
      pulled: pulled.value,
      lastPulled: lastPulled.value,
    }),
  );
};

const loadFromLocalStorage = () => {
  const savedData = localStorage.getItem(STORAGE_KEY);

  if (!savedData) return;

  try {
    const parsedData = JSON.parse(savedData);

    if (
      parsedData.partyType === 'girls' ||
      parsedData.partyType === 'boys'
    ) {
      partyType.value = parsedData.partyType;
    }

    if (parsedData.diapason) {
      diapason.value = parsedData.diapason;
    }

    if (Array.isArray(parsedData.pulled)) {
      pulled.value = parsedData.pulled;
    }

    if (
      typeof parsedData.lastPulled === 'number' ||
      parsedData.lastPulled === null
    ) {
      lastPulled.value = parsedData.lastPulled;
    }
  } catch (error) {
    console.error(
      'Не удалось загрузить данные из localStorage',
      error,
    );
  }
};

const generateRandom = () => {
  if (!canGenerate.value || isCountingDown.value) return;

  const min = Number(diapason.value.min);
  const max = Number(diapason.value.max);

  if (
    !Number.isFinite(min) ||
    !Number.isFinite(max) ||
    min > max
  ) {
    return;
  }

  let randomValue;

  do {
    randomValue =
      Math.floor(Math.random() * (max - min + 1)) + min;
  } while (pulled.value.includes(randomValue));

  lastPulled.value = randomValue;
  pulled.value.push(randomValue);

  showHint.value = false;
  showAnswer.value = false;
  showHintModal.value = false;
  showAnswerModal.value = false;

  startCountdown(randomValue);
};

const clear = () => {
  const confirmed = window.confirm(
    'Вы уверены, что хотите сбросить все выпавшие числа?',
  );

  if (!confirmed) return;

  pulled.value = [];
  lastPulled.value = null;
};

onMounted(() => {
  loadFromLocalStorage();
});

watch(
  [partyType, diapason, pulled, lastPulled],
  saveToLocalStorage,
  {
    deep: true,
  },
);

onBeforeUnmount(() => {
  destroyAudio();
});
</script>

<template>
  <main class="app" :class="`theme-${partyType}`">
    <LoginForm v-if="!isAuthorized" @success="handleLoginSuccess" />
    <template v-else>
      <Transition name="countdown">
        <div v-if="isCountingDown" class="countdown-overlay">
          <div :key="countdown" class="countdown-number">
            {{ countdown }}
          </div>

          <div class="countdown-label">
            УГАДАЙ ПЕСНЮ
          </div>
        </div>
      </Transition>
      <div class="content">

        <!-- =========================
           ПЕРЕКЛЮЧАТЕЛЬ
      ========================== -->

        <div class="party-switch">
          <button type="button" class="party-switch-button" :class="{ active: partyType === 'girls' }"
            @click="setPartyType('girls')">
            <span class="switch-icon">
              ♡
            </span>

            Девичник
          </button>

          <button type="button" class="party-switch-button" :class="{ active: partyType === 'boys' }"
            @click="setPartyType('boys')">
            <span class="switch-icon">
              ♠
            </span>

            Мальчишник
          </button>
        </div>

        <!-- =========================
           ГЕНЕРАТОР
      ========================== -->

        <section class="generator">

          <!-- HEADER -->

          <header class="header">
            <h1>
              МУЗЛОТО
            </h1>
            <div class="logo-wrapper">
              <img src="/free-icon-lotto-4994219.png" alt="Лото" class="logo">
            </div>
          </header>

          <!-- =========================
             ДИАПАЗОН
        ========================== -->

          <div class="range">
            <label>
              <span>
                От
              </span>

              <input v-model.number="diapason.min" @change="(el) => { P }" min="0" type="number">
            </label>

            <span class="range-divider">
              —
            </span>

            <label>
              <span>
                До
              </span>

              <input v-model.number="diapason.max" type="number" max="100">
            </label>
          </div>

          <!-- =========================
             РЕЗУЛЬТАТ
        ========================== -->

          <div class="result">
            <div class="number" :class="{ empty: lastPulled === null }">
              {{ lastPulled ?? '?' }}
            </div>

            <span class="result-label">
              {{
                lastPulled === null
                  ? 'Готовы? Жмите!'
                  : 'Последнее число'
              }}
            </span>
          </div>

          <!-- =========================
     MUSIC PLAYER
========================== -->

          <div v-if="lastPulled !== null" class="music-player">

            <!-- Верхняя строка -->

            <div class="music-header">

              <div class="playing-status">

                <!-- АНИМАЦИЯ ЗВУКА -->

                <div class="equalizer" :class="{
                  active: isPlaying,
                }">
                  <span></span>
                  <span></span>
                  <span></span>
                  <span></span>
                  <span></span>
                </div>

                <div class="music-status-text">
                  <strong>
                    {{
                      audioFinished
                        ? 'Фрагмент закончился'
                        : isPlaying
                          ? 'Сейчас играет'
                          : 'На паузе'
                    }}
                  </strong>

                  <span>
                    Трек №{{ lastPulled }}
                  </span>
                </div>

              </div>

              <div class="music-time">
                {{ formatTime(currentTime) }}
                /
                {{ formatTime(duration) }}
              </div>

            </div>


            <!-- PROGRESS -->

            <div class="progress-track">
              <div class="progress-value" :style="{
                width: `${progress}%`,
              }"></div>
            </div>


            <!-- CONTROLS -->

            <div class="music-controls">

              <button type="button" class="music-control primary-control" :disabled="audioFinished"
                @click="togglePause">
                <template v-if="isPlaying">
                  ❚❚ Пауза
                </template>

                <template v-else>
                  ▶ Продолжить
                </template>
              </button>


              <button type="button" class="music-control" @click="replaySong">
                ↻ Заново
              </button>


              <!-- VOLUME -->

              <div class="volume-control">
                <span>
                  🔊
                </span>

                <input v-model.number="volume" type="range" min="0" max="1" step="0.05" @input="changeVolume">
              </div>

            </div>


            <!-- HINT / ANSWER -->

            <div class="game-help">

              <div class="game-help-buttons">

                <button type="button" class="hint-button" @click="showHintModal = true">
                  💡 Подсказка
                </button>

                <button type="button" class="answer-button" @click="showAnswerModal = true">
                  👀 Показать ответ
                </button>
              </div>


              <!-- ПОДСКАЗКА -->

              <Transition name="help">
                <div v-if="showHint" class="help-content hint-content">
                  <span class="help-label">
                    ПОДСКАЗКА
                  </span>

                  <p>
                    {{ currentSong.hint }}
                  </p>
                </div>
              </Transition>


              <!-- ОТВЕТ -->

              <Transition name="help">
                <div v-if="showAnswer" class="help-content answer-content">
                  <span class="help-label">
                    ОТВЕТ
                  </span>

                  <strong>
                    {{ currentSong.artist }}
                  </strong>

                  <p>
                    {{ currentSong.title }}
                  </p>
                </div>
              </Transition>

            </div>

          </div>

          <!-- =========================
             КНОПКА
        ========================== -->

          <button v-if="!isFinished" type="button" class="generate" :disabled="!canGenerate" @click="generateRandom">
            <span class="generate-icon">
              ◆
            </span>

            {{
              audioFinished
                ? 'Сгенерировать число'
                : 'Сначала дослушайте текущий трек'
            }}
          </button>

          <div v-else class="finished">
            🎉 Все числа выпали
          </div>

          <!-- =========================
             ИСТОРИЯ
        ========================== -->

          <div class="history">
            <div class="history-header">
              <h2>
                Выпавшие числа
              </h2>

              <span>
                {{ pulled.length }}
                /
                {{ totalNumbers }}
              </span>
            </div>

            <div v-if="pulled.length" class="numbers">
              <div v-for="(value, index) in pulled" :key="value" class="history-number" :class="{
                latest:
                  index === pulled.length - 1,
              }">
                {{ value }}
              </div>
            </div>

            <div v-else class="empty-history">
              Здесь появятся выпавшие числа
            </div>
          </div>

          <!-- =========================
             НИЗ
        ========================== -->

          <div class="bottom">
            <button v-if="pulled.length" type="button" class="reset" @click="clear">
              <span>
                ↻
              </span>

              Сбросить
            </button>

            <div class="credits">
              <a href="https://www.flaticon.com/ru/free-icons/" target="_blank" rel="noopener noreferrer"
                title="лото иконки">
                Lotto icon by Magnific — Flaticon
              </a>
            </div>
          </div>

        </section>
      </div>
      <Transition name="modal">
        <div v-if="showHintModal" class="game-modal" @click.self="showHintModal = false">
          <div class="game-modal-card hint-modal-card">

            <button type="button" class="modal-close" @click="showHintModal = false">
              ×
            </button>

            <div class="modal-icon">
              💡
            </div>

            <div class="modal-label">
              ПОДСКАЗКА
            </div>

            <div class="modal-main-text">
              {{ currentSong?.hint }}
            </div>

            <button type="button" class="modal-action" @click="showHintModal = false">
              Понятно
            </button>

          </div>
        </div>
      </Transition>
      <Transition name="modal">
        <div v-if="showAnswerModal" class="game-modal" @click.self="showAnswerModal = false">
          <div class="game-modal-card answer-modal-card">

            <button type="button" class="modal-close" @click="showAnswerModal = false">
              ×
            </button>

            <div class="modal-icon">
              🎵
            </div>

            <div class="modal-label">
              ПРАВИЛЬНЫЙ ОТВЕТ
            </div>

            <div class="answer-artist">
              {{ currentSong?.artist }}
            </div>

            <div class="answer-title">
              {{ currentSong?.title }}
            </div>

            <button type="button" class="modal-action" @click="showAnswerModal = false">
              Закрыть
            </button>

          </div>
        </div>
      </Transition>
    </template>
  </main>

</template>

<style>
/* =====================================================
   RESET
===================================================== */

:root {
  font-family:
    Inter,
    -apple-system,
    BlinkMacSystemFont,
    "Segoe UI",
    sans-serif;
}

html,
body,
#app {
  width: 100%;
  height: 100%;
  margin: 0;
}

html,
body {
  overflow: hidden;
}

*,
*::before,
*::after {
  box-sizing: border-box;
}

button,
input {
  font: inherit;
}

button {
  cursor: pointer;
}


/* =====================================================
   APP / BACKGROUND
===================================================== */

.app {
  --accent: #ff3f91;
  --accent-light: #ff78b3;
  --accent-dark: #d41e68;

  --accent-rgb: 255, 63, 145;

  width: 100vw;
  height: 100dvh;

  display: flex;
  align-items: center;
  justify-content: center;

  padding: 14px;

  overflow: hidden;

  color: #ffffff;

  background-image:
    linear-gradient(rgba(0, 0, 0, 0.08),
      rgba(0, 0, 0, 0.08)),
    url('/party-background.jpeg');

  background-position: center;
  background-repeat: no-repeat;
  background-size: cover;
}


/* =====================================================
   THEMES
===================================================== */

.app.theme-girls {
  --accent: #ff3f91;
  --accent-light: #ff78b3;
  --accent-dark: #d41e68;

  --accent-rgb: 255, 63, 145;
}

.app.theme-boys {
  --accent: #748cff;
  --accent-light: #a1b0ff;
  --accent-dark: #4056d8;

  --accent-rgb: 116, 140, 255;
}


/* =====================================================
   CONTENT
===================================================== */

.content {
  width: 100%;
  max-width: 520px;

  display: flex;
  flex-direction: column;

  gap: 9px;

  position: relative;
  z-index: 5;
}


/* =====================================================
   PARTY SWITCH
===================================================== */

.party-switch {
  width: 390px;
  max-width: 100%;

  margin: 0 auto;

  display: grid;
  grid-template-columns: 1fr 1fr;

  padding: 4px;

  border:
    1px solid rgba(255, 255, 255, 0.12);

  border-radius: 17px;

  background:
    rgba(8, 9, 14, 0.92);

  backdrop-filter: blur(18px);

  box-shadow:
    0 12px 35px rgba(0, 0, 0, 0.42);
}

.party-switch-button {
  height: 45px;

  display: flex;
  align-items: center;
  justify-content: center;

  gap: 7px;

  padding: 5px 12px;

  border: none;
  border-radius: 13px;

  color:
    rgba(255, 255, 255, 0.48);

  background: transparent;

  font-size: 13px;
  font-weight: 700;

  transition:
    0.2s ease;
}

.party-switch-button:hover {
  color: #ffffff;
}

.party-switch-button.active {
  color: #ffffff;

  background:
    linear-gradient(135deg,
      var(--accent-light),
      var(--accent));

  box-shadow:
    0 6px 20px rgba(var(--accent-rgb), 0.35);
}

.switch-icon {
  font-size: 16px;
}


/* =====================================================
   GENERATOR CARD
===================================================== */

.generator {
  position: relative;

  width: 100%;

  padding:
    20px 24px 13px;

  overflow: hidden;

  border:
    1px solid rgba(255, 255, 255, 0.12);

  border-radius: 25px;

  background:
    linear-gradient(145deg,
      rgba(24, 24, 33, 0.95),
      rgba(7, 8, 13, 0.97));

  backdrop-filter: blur(24px);

  box-shadow:
    0 35px 90px rgba(0, 0, 0, 0.6),
    inset 0 1px 0 rgba(255, 255, 255, 0.06);
}

.generator::before {
  content: "";

  position: absolute;

  top: -190px;
  left: 50%;

  width: 300px;
  height: 300px;

  transform: translateX(-50%);

  border-radius: 50%;

  background:
    rgba(var(--accent-rgb), 0.22);

  filter: blur(65px);

  pointer-events: none;
}


/* =====================================================
   HEADER
===================================================== */

.header {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.logo-wrapper {
  width: 42px;
  height: 42px;

  flex: 0 0 42px;

  display: flex;
  align-items: center;
  justify-content: center;

  /* ВАЖНО — никаких margin: 0 auto */
  margin: 0;

  border: 1px solid rgba(var(--accent-rgb), 0.3);
  border-radius: 12px;

  background: rgba(var(--accent-rgb), 0.12);
}

.logo {
  width: 27px;
  height: 27px;

  display: block;

  object-fit: contain;
}

.eyebrow {
  display: block;

  color:
    var(--accent-light);

  font-size: 8px;
  font-weight: 800;

  letter-spacing: 0.18em;
}

.header h1 {
  margin: 0;
  font-size: 32px;
  font-weight: 900;
  line-height: 1;
  letter-spacing: 0.04em;
}

.header p {
  margin: 0;

  color:
    rgba(255, 255, 255, 0.42);

  font-size: 11px;
}


/* =====================================================
   RANGE
===================================================== */

.range {
  display: flex;

  justify-content: center;
  align-items: flex-end;

  gap: 9px;

  margin:
    12px 0 9px;
}

.range label {
  display: flex;
  flex-direction: column;

  gap: 4px;

  color:
    rgba(255, 255, 255, 0.55);

  font-size: 9px;
}

.range input {
  width: 82px;
  height: 36px;

  padding: 5px 8px;

  border:
    1px solid rgba(255, 255, 255, 0.12);

  border-radius: 10px;

  outline: none;

  color: #ffffff;

  background:
    rgba(255, 255, 255, 0.05);

  text-align: center;

  font-size: 15px;
  font-weight: 700;
}

.range input:focus {
  border-color:
    rgba(var(--accent-rgb), 0.7);

  box-shadow:
    0 0 0 3px rgba(var(--accent-rgb), 0.09);
}

.range-divider {
  height: 36px;

  display: flex;
  align-items: center;

  color:
    rgba(255, 255, 255, 0.35);
}


/* =====================================================
   RESULT
===================================================== */

.result {
  display: flex;
  flex-direction: column;
  align-items: center;

  margin:
    2px 0 10px;
}

.number {
  position: relative;

  width: 108px;
  height: 108px;

  display: flex;
  align-items: center;
  justify-content: center;

  border-radius: 50%;

  color: #ffffff;

  background:
    radial-gradient(circle at 35% 25%,
      var(--accent-light),
      var(--accent) 52%,
      var(--accent-dark));

  border:
    1px solid rgba(255, 255, 255, 0.24);

  font-size: 47px;
  font-weight: 900;

  line-height: 1;

  box-shadow:
    0 0 40px rgba(var(--accent-rgb), 0.35),
    inset 0 2px 7px rgba(255, 255, 255, 0.22);
}

.number::after {
  content: "";

  position: absolute;

  inset: 8px;

  border:
    1px solid rgba(255, 255, 255, 0.12);

  border-radius: inherit;

  pointer-events: none;
}

.number.empty {
  color:
    rgba(255, 255, 255, 0.22);

  background:
    rgba(255, 255, 255, 0.025);

  border:
    1px dashed rgba(255, 255, 255, 0.13);

  box-shadow: none;
}

.result-label {
  margin-top: 5px;

  color:
    rgba(255, 255, 255, 0.4);

  font-size: 9px;
}


/* =====================================================
   GENERATE
===================================================== */

.generate {
  width: 100%;
  min-height: 42px;

  display: flex;
  align-items: center;
  justify-content: center;

  gap: 7px;

  padding:
    8px 15px;

  border: none;
  border-radius: 11px;

  color: #ffffff;

  background:
    linear-gradient(105deg,
      var(--accent-light),
      var(--accent),
      var(--accent-dark));

  font-size: 13px;
  font-weight: 800;

  box-shadow:
    0 10px 28px rgba(var(--accent-rgb), 0.26);

  transition:
    0.2s ease;
}

.generate:hover {
  transform:
    translateY(-1px);

  box-shadow:
    0 13px 35px rgba(var(--accent-rgb), 0.38);
}

.generate:active {
  transform:
    scale(0.985);
}

.generate-icon {
  font-size: 9px;
}

.finished {
  min-height: 42px;

  display: flex;
  align-items: center;
  justify-content: center;

  border:
    1px solid rgba(var(--accent-rgb), 0.27);

  border-radius: 11px;

  color:
    var(--accent-light);

  background:
    rgba(var(--accent-rgb), 0.075);

  font-size: 13px;
  font-weight: 700;
}

.generate:disabled {
  cursor: not-allowed;

  opacity: 0.4;

  filter: grayscale(0.35);

  transform: none;

  box-shadow: none;
}

/* =====================================================
   HISTORY
===================================================== */

.history {
  margin-top: 10px;

  padding-top: 8px;

  border-top:
    1px solid rgba(255, 255, 255, 0.075);
}

.history-header {
  display: flex;
  align-items: center;
  justify-content: space-between;

  margin-bottom: 7px;
}

.history-header h2 {
  margin: 0;

  font-size: 10px;
  font-weight: 700;
}

.history-header span {
  color:
    rgba(255, 255, 255, 0.37);

  font-size: 9px;
}


/* =====================================================
   HISTORY NUMBERS
===================================================== */

.numbers {
  display: flex;
  flex-wrap: wrap;

  align-items: center;

  gap: 6px;
}

.history-number {
  flex: 0 0 auto;

  width: 34px;
  height: 34px;

  display: flex;
  align-items: center;
  justify-content: center;

  border:
    1px solid rgba(255, 255, 255, 0.09);

  border-radius: 50%;

  color:
    rgba(255, 255, 255, 0.68);

  background:
    rgba(255, 255, 255, 0.032);

  font-size: 10px;
  font-weight: 700;

  transition:
    0.2s ease;
}

.history-number.latest {
  color: #ffffff;

  background:
    var(--accent);

  border-color:
    var(--accent);

  box-shadow:
    0 0 16px rgba(var(--accent-rgb), 0.42);
}

.empty-history {
  padding: 8px;

  border:
    1px dashed rgba(255, 255, 255, 0.08);

  border-radius: 9px;

  color:
    rgba(255, 255, 255, 0.25);

  text-align: center;

  font-size: 9px;
}


/* =====================================================
   RESET / CREDITS
===================================================== */

.bottom {
  text-align: center;
}

.reset {
  display: inline-flex;
  align-items: center;
  justify-content: center;

  gap: 4px;

  margin-top: 7px;

  padding:
    4px 9px;

  border: none;

  color:
    rgba(255, 255, 255, 0.38);

  background: none;

  font-size: 9px;

  transition:
    0.2s ease;
}

.reset:hover {
  color: #ffffff;
}

.credits {
  margin-top: 1px;
}

.credits a {
  color:
    rgba(255, 255, 255, 0.16);

  text-decoration: none;

  font-size: 6px;
}

.credits a:hover {
  color:
    rgba(255, 255, 255, 0.5);
}


/* =====================================================
   MUSIC PLAYER
===================================================== */

.music-player {
  margin-bottom: 10px;

  padding: 13px;

  border:
    1px solid rgba(255, 255, 255, 0.09);

  border-radius: 14px;

  background:
    linear-gradient(145deg,
      rgba(255, 255, 255, 0.055),
      rgba(255, 255, 255, 0.02));
}


/* HEADER */

.music-header {
  display: flex;
  align-items: center;
  justify-content: space-between;

  gap: 15px;
}

.playing-status {
  display: flex;
  align-items: center;

  gap: 10px;
}

.music-status-text {
  display: flex;
  flex-direction: column;

  gap: 2px;
}

.music-status-text strong {
  font-size: 11px;
}

.music-status-text span {
  color:
    rgba(255, 255, 255, 0.38);

  font-size: 8px;
}

.music-time {
  color:
    rgba(255, 255, 255, 0.36);

  font-size: 8px;

  white-space: nowrap;
}


/* =====================================================
   EQUALIZER
===================================================== */

.equalizer {
  width: 29px;
  height: 25px;

  display: flex;
  align-items: center;
  justify-content: center;

  gap: 2px;
}

.equalizer span {
  width: 3px;
  height: 5px;

  border-radius: 3px;

  background:
    var(--accent);

  box-shadow:
    0 0 8px rgba(var(--accent-rgb), 0.4);

  transition:
    height 0.2s ease;
}

.equalizer.active span {
  animation:
    sound-wave 0.75s ease-in-out infinite alternate;
}

.equalizer.active span:nth-child(2) {
  animation-delay: -0.2s;
}

.equalizer.active span:nth-child(3) {
  animation-delay: -0.4s;
}

.equalizer.active span:nth-child(4) {
  animation-delay: -0.1s;
}

.equalizer.active span:nth-child(5) {
  animation-delay: -0.3s;
}

@keyframes sound-wave {
  0% {
    height: 5px;
  }

  50% {
    height: 20px;
  }

  100% {
    height: 9px;
  }
}


/* =====================================================
   PROGRESS
===================================================== */

.progress-track {
  width: 100%;
  height: 4px;

  margin:
    10px 0;

  overflow: hidden;

  border-radius: 10px;

  background:
    rgba(255, 255, 255, 0.08);
}

.progress-value {
  height: 100%;

  border-radius: inherit;

  background:
    linear-gradient(90deg,
      var(--accent-light),
      var(--accent));

  box-shadow:
    0 0 10px rgba(var(--accent-rgb), 0.45);

  transition:
    width 0.15s linear;
}


/* =====================================================
   CONTROLS
===================================================== */

.music-controls {
  display: grid;

  grid-template-columns:
    auto auto 1fr;

  align-items: center;

  gap: 7px;
}

.music-control {
  height: 32px;

  padding:
    0 11px;

  border:
    1px solid rgba(255, 255, 255, 0.09);

  border-radius: 8px;

  color:
    rgba(255, 255, 255, 0.75);

  background:
    rgba(255, 255, 255, 0.04);

  font-size: 9px;
  font-weight: 700;

  transition:
    0.2s ease;
}

.music-control:hover:not(:disabled) {
  color: #fff;

  background:
    rgba(255, 255, 255, 0.08);
}

.music-control:disabled {
  cursor: default;

  opacity: 0.3;
}

.primary-control {
  color: #fff;

  border-color:
    rgba(var(--accent-rgb), 0.35);

  background:
    rgba(var(--accent-rgb), 0.12);
}


/* =====================================================
   VOLUME
===================================================== */

.volume-control {
  display: flex;
  align-items: center;

  gap: 6px;

  padding-left: 5px;
}

.volume-control span {
  font-size: 11px;
}

.volume-control input {
  width: 100%;
  min-width: 70px;

  accent-color:
    var(--accent);

  cursor: pointer;
}


/* =====================================================
   HINT / ANSWER
===================================================== */

.game-help {
  margin-top: 10px;

  padding-top: 9px;

  border-top:
    1px solid rgba(255, 255, 255, 0.07);
}

.game-help-buttons {
  display: grid;

  grid-template-columns:
    1fr 1fr;

  gap: 8px;

  margin-top: 10px;

  padding-top: 10px;

  border-top:
    1px solid rgba(255, 255, 255, 0.07);
}

.hint-button,
.answer-button {
  min-height: 38px;

  border:
    1px solid rgba(255, 255, 255, 0.1);

  border-radius: 10px;

  color: #fff;

  background:
    rgba(255, 255, 255, 0.05);

  font-size: 11px;
  font-weight: 700;
}

.hint-button:hover,
.answer-button:hover {
  border-color:
    rgba(var(--accent-rgb), 0.4);

  background:
    rgba(var(--accent-rgb), 0.12);
}


/* =====================================================
   HELP CONTENT
===================================================== */

.help-content {
  margin-top: 7px;

  padding:
    9px 11px;

  border-radius: 8px;

  text-align: left;
}

.help-label {
  display: block;

  margin-bottom: 3px;

  font-size: 6px;
  font-weight: 900;

  letter-spacing: 0.13em;
}

.help-content p {
  margin: 0;

  font-size: 10px;
  line-height: 1.35;
}

.hint-content {
  color:
    rgba(255, 255, 255, 0.83);

  border:
    1px solid rgba(255, 200, 72, 0.13);

  background:
    rgba(255, 200, 72, 0.055);
}

.hint-content .help-label {
  color: #f4c34e;
}

.answer-content {
  border:
    1px solid rgba(var(--accent-rgb), 0.18);

  background:
    rgba(var(--accent-rgb), 0.07);
}

.answer-content .help-label {
  color:
    var(--accent-light);
}

.answer-content strong {
  display: block;

  margin-bottom: 2px;

  font-size: 11px;
}

.answer-content p {
  color:
    rgba(255, 255, 255, 0.58);
}


/* =====================================================
   ANIMATION
===================================================== */

.help-enter-active,
.help-leave-active {
  transition:
    opacity 0.2s ease,
    transform 0.2s ease;
}

.help-enter-from,
.help-leave-to {
  opacity: 0;

  transform:
    translateY(-5px);
}


/* =====================================================
   MOBILE
===================================================== */

@media (max-width: 700px) {
  .music-player {
    padding: 8px;

    margin-bottom: 5px;

    border-radius: 10px;
  }

  .music-header {
    gap: 7px;
  }

  .equalizer {
    width: 23px;
    height: 20px;
  }

  .music-controls {
    grid-template-columns:
      1fr 1fr;

    gap: 5px;
  }

  .volume-control {
    grid-column:
      1 / -1;
  }

  .music-control {
    height: 28px;

    font-size: 8px;
  }

  .game-help {
    margin-top: 6px;

    padding-top: 6px;
  }

  .hint-button,
  .answer-button {
    min-height: 27px;

    font-size: 8px;
  }

  .help-content {
    padding:
      6px 8px;
  }

  .help-content p {
    font-size: 8px;
  }

  .answer-content strong {
    font-size: 9px;
  }
}

/* =====================================================
   PROJECTOR / 16:9
===================================================== */

@media (min-width: 1200px) {
  .content {
    max-width: 520px;
  }
}


/* =====================================================
   LOWER HEIGHT
===================================================== */

@media (max-height: 800px) and (min-width: 701px) {
  .content {
    gap: 6px;
  }

  .party-switch-button {
    height: 39px;
  }

  .generator {
    padding:
      13px 21px 8px;
  }

  .logo-wrapper {
    width: 42px;
    height: 42px;

    display: flex;
    align-items: center;
    justify-content: center;

    border: 1px solid rgba(var(--accent-rgb), 0.3);
    border-radius: 12px;

    background: rgba(var(--accent-rgb), 0.12);
  }

  .logo {
    width: 27px;
    height: 27px;
    object-fit: contain;
  }

  .header h1 {
    font-size: 22px;
  }

  .header p {
    font-size: 9px;
  }

  .range {
    margin:
      7px 0 5px;
  }

  .range input {
    height: 32px;
  }

  .range-divider {
    height: 32px;
  }

  .number {
    width: 86px;
    height: 86px;

    font-size: 37px;
  }

  .result {
    margin-bottom: 5px;
  }

  .generate,
  .finished {
    min-height: 35px;
  }

  .history {
    margin-top: 6px;
    padding-top: 5px;
  }

  .history-header {
    margin-bottom: 4px;
  }

  .numbers {
    gap: 4px;
  }

  .history-number {
    width: 29px;
    height: 29px;

    font-size: 8px;
  }

  .reset {
    margin-top: 3px;
  }
}

/* =====================================================
   MODAL
===================================================== */
.game-modal {
  position: fixed;
  inset: 0;
  z-index: 1100;

  display: flex;
  align-items: center;
  justify-content: center;

  padding: 40px;

  background:
    rgba(2, 3, 8, 0.84);

  backdrop-filter:
    blur(16px);
}

.game-modal-card {
  position: relative;

  width: min(900px, 90vw);

  min-height: 430px;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  padding:
    60px 70px;

  border:
    1px solid rgba(255, 255, 255, 0.14);

  border-radius: 34px;

  text-align: center;

  background:
    linear-gradient(145deg,
      rgba(27, 27, 39, 0.98),
      rgba(8, 8, 14, 0.98));

  box-shadow:
    0 40px 120px rgba(0, 0, 0, 0.7),
    0 0 80px rgba(var(--accent-rgb), 0.13);
}

.game-modal-card::before {
  content: "";

  position: absolute;

  inset: 0;

  border-radius: inherit;

  background:
    radial-gradient(circle at 50% 0%,
      rgba(var(--accent-rgb), 0.16),
      transparent 48%);

  pointer-events: none;
}

.modal-close {
  position: absolute;

  top: 20px;
  right: 24px;

  width: 48px;
  height: 48px;

  display: flex;
  align-items: center;
  justify-content: center;

  border:
    1px solid rgba(255, 255, 255, 0.1);

  border-radius: 50%;

  color:
    rgba(255, 255, 255, 0.65);

  background:
    rgba(255, 255, 255, 0.05);

  font-size: 28px;
  line-height: 1;
}

.modal-close:hover {
  color: #fff;

  background:
    rgba(255, 255, 255, 0.1);
}

.modal-icon {
  position: relative;
  z-index: 1;

  margin-bottom: 16px;

  font-size: 64px;
}

.modal-label {
  position: relative;
  z-index: 1;

  margin-bottom: 25px;

  color:
    var(--accent-light);

  font-size: 16px;
  font-weight: 900;

  letter-spacing: 0.2em;
}

.modal-main-text {
  position: relative;
  z-index: 1;

  max-width: 750px;

  color: #fff;

  font-size:
    clamp(32px, 4vw, 58px);

  font-weight: 800;

  line-height: 1.18;
}

.answer-artist {
  position: relative;
  z-index: 1;

  color:
    rgba(255, 255, 255, 0.62);

  font-size:
    clamp(24px, 3vw, 38px);

  font-weight: 700;
}

.answer-title {
  position: relative;
  z-index: 1;

  margin-top: 10px;

  color: #fff;

  font-size:
    clamp(42px, 5vw, 72px);

  font-weight: 900;

  line-height: 1.05;
}

.modal-action {
  position: relative;
  z-index: 1;

  min-width: 190px;
  min-height: 50px;

  margin-top: 38px;

  padding:
    12px 28px;

  border: none;
  border-radius: 14px;

  color: #fff;

  background:
    linear-gradient(105deg,
      var(--accent-light),
      var(--accent),
      var(--accent-dark));

  font-size: 15px;
  font-weight: 800;

  box-shadow:
    0 10px 30px rgba(var(--accent-rgb), 0.32);
}

.modal-enter-active,
.modal-leave-active {
  transition:
    opacity 0.2s ease;
}

.modal-enter-active .game-modal-card,
.modal-leave-active .game-modal-card {
  transition:
    transform 0.25s ease,
    opacity 0.25s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .game-modal-card {
  opacity: 0;

  transform:
    scale(0.9) translateY(20px);
}

.modal-leave-to .game-modal-card {
  opacity: 0;

  transform:
    scale(0.95);
}


/* =====================================================
   countdown
===================================================== */
.countdown-overlay {
  position: fixed;
  inset: 0;
  z-index: 1000;

  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;

  background:
    rgba(4, 4, 8, 0.78);

  backdrop-filter: blur(12px);
}

.countdown-number {
  color: #fff;

  font-size: clamp(140px, 20vw, 280px);
  font-weight: 900;

  line-height: 0.9;

  text-shadow:
    0 0 30px rgba(var(--accent-rgb), 0.8),
    0 0 90px rgba(var(--accent-rgb), 0.45);

  animation:
    countdown-pop 0.95s ease both;
}

.countdown-label {
  margin-top: 28px;

  color:
    rgba(255, 255, 255, 0.72);

  font-size: 20px;
  font-weight: 800;

  letter-spacing: 0.28em;
}

@keyframes countdown-pop {
  0% {
    opacity: 0;

    transform:
      scale(1.6);
  }

  25% {
    opacity: 1;

    transform:
      scale(1);
  }

  75% {
    opacity: 1;

    transform:
      scale(1);
  }

  100% {
    opacity: 0;

    transform:
      scale(0.7);
  }
}

.countdown-enter-active,
.countdown-leave-active {
  transition:
    opacity 0.2s ease;
}

.countdown-enter-from,
.countdown-leave-to {
  opacity: 0;
}

/* =====================================================
   MOBILE
===================================================== */

@media (max-width: 700px) {
  .app {
    padding: 6px;

    background-position: center;
  }

  .content {
    max-width: 430px;

    gap: 5px;
  }

  .party-switch {
    width: 100%;

    border-radius: 14px;
  }

  .party-switch-button {
    height: 37px;

    padding: 4px 8px;

    border-radius: 10px;

    font-size: 10px;
  }

  .switch-icon {
    font-size: 13px;
  }

  .generator {
    padding:
      10px 12px 7px;

    border-radius: 18px;
  }

  .logo-wrapper {
    width: 31px;
    height: 31px;

    margin-bottom: 3px;

    border-radius: 9px;
  }

  .logo {
    width: 19px;
    height: 19px;
  }

  .eyebrow {
    font-size: 6px;
  }

  .header h1 {
    margin:
      2px 0 0;

    font-size: 19px;
  }

  .header p {
    font-size: 8px;
  }

  .range {
    gap: 6px;

    margin:
      6px 0 4px;
  }

  .range label {
    gap: 2px;

    font-size: 7px;
  }

  .range input {
    width: 65px;
    height: 29px;

    padding: 2px 4px;

    border-radius: 8px;

    font-size: 12px;
  }

  .range-divider {
    height: 29px;

    font-size: 11px;
  }

  .result {
    margin:
      1px 0 5px;
  }

  .number {
    width: 77px;
    height: 77px;

    font-size: 33px;
  }

  .number::after {
    inset: 6px;
  }

  .result-label {
    margin-top: 3px;

    font-size: 7px;
  }

  .generate,
  .finished {
    min-height: 33px;

    border-radius: 8px;

    font-size: 10px;
  }

  .history {
    margin-top: 5px;
    padding-top: 4px;
  }

  .history-header {
    margin-bottom: 4px;
  }

  .history-header h2 {
    font-size: 8px;
  }

  .history-header span {
    font-size: 7px;
  }

  .numbers {
    gap: 3px;
  }

  .history-number {
    width: 27px;
    height: 27px;

    font-size: 8px;
  }

  .empty-history {
    padding: 5px;

    font-size: 7px;
  }

  .reset {
    margin-top: 2px;

    padding:
      2px 6px;

    font-size: 7px;
  }

  .credits {
    margin-top: 0;
  }

  .credits a {
    font-size: 5px;
  }
}
</style>