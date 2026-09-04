<script setup>
import { computed, onMounted, ref, watch } from 'vue';

const STORAGE_KEY = 'random-number-generator';

const partyType = ref('girls');

const diapason = ref({
  min: 1,
  max: 42,
});

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
  if (isFinished.value) return;

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
</script>

<template>
  <main
    class="app"
    :class="`theme-${partyType}`"
  >
    <!-- ===============================
         BACKGROUND
    ================================ -->

    <div class="ambient ambient-left"></div>
    <div class="ambient ambient-right"></div>

    <!-- Декоративные иконки -->

    <span
      class="party-decoration decor-heart"
      aria-hidden="true"
    >
      ♥
    </span>

    <span
      class="party-decoration decor-sparkle"
      aria-hidden="true"
    >
      ✦
    </span>

    <span
      class="party-decoration decor-heart-outline"
      aria-hidden="true"
    >
      ♡
    </span>

    <span
      class="party-decoration decor-spade"
      aria-hidden="true"
    >
      ♠
    </span>

    <span
      class="party-decoration decor-crown"
      aria-hidden="true"
    >
      ♛
    </span>

    <span
      class="party-decoration decor-club"
      aria-hidden="true"
    >
      ♣
    </span>

    <!-- Надписи -->

    <div
      class="party-text party-text-left"
      aria-hidden="true"
    >
      TEAM<br>
      BRIDE
    </div>

    <div
      class="party-text party-text-right"
      aria-hidden="true"
    >
      TEAM<br>
      GROOM
    </div>

    <!-- ===============================
         НАСТЯ
    ================================ -->

    <div
      class="person person-nastya"
      aria-hidden="true"
    >
      <img
        src="/nastya.png"
        alt=""
      >
    </div>

    <!-- ===============================
         ВАНЯ
    ================================ -->

    <div
      class="person person-vanya"
      aria-hidden="true"
    >
      <img
        src="/vanya.png"
        alt=""
      >
    </div>

    <!-- ===============================
         ОСНОВНОЙ КОНТЕНТ
    ================================ -->

    <div class="content">

      <!-- PARTY SWITCH -->

      <div class="party-switch">
        <button
          type="button"
          class="party-switch-button"
          :class="{
            active: partyType === 'girls',
          }"
          @click="setPartyType('girls')"
        >
          <span class="switch-icon">
            ♡
          </span>

          Девичник
        </button>

        <button
          type="button"
          class="party-switch-button"
          :class="{
            active: partyType === 'boys',
          }"
          @click="setPartyType('boys')"
        >
          <span class="switch-icon">
            ♠
          </span>

          Мальчишник
        </button>
      </div>

      <!-- ===============================
           GENERATOR CARD
      ================================ -->

      <section class="generator">

        <!-- HEADER -->

        <header class="header">
          <div class="logo-wrapper">
            <img
              src="/free-icon-lotto-4994219.png"
              alt="Лото"
              class="logo"
            >
          </div>

          <span class="eyebrow">
            {{ partyLabel }}
          </span>

          <h1>
            Музыкальное лото
          </h1>
        </header>

        <!-- RANGE -->

        <div class="range">
          <label>
            <span>
              От
            </span>

            <input
              v-model.number="diapason.min"
              type="number"
            >
          </label>

          <span class="range-divider">
            —
          </span>

          <label>
            <span>
              До
            </span>

            <input
              v-model.number="diapason.max"
              type="number"
            >
          </label>
        </div>

        <!-- LAST NUMBER -->

        <div class="result">
          <div
            class="number"
            :class="{
              empty: lastPulled === null,
            }"
          >
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

        <!-- GENERATE -->

        <button
          v-if="!isFinished"
          type="button"
          class="generate"
          @click="generateRandom"
        >
          <span class="generate-icon">
            ◆
          </span>

          Сгенерировать число
        </button>

        <div
          v-else
          class="finished"
        >
          🎉 Все числа выпали
        </div>

        <!-- HISTORY -->

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

          <div
            v-if="pulled.length"
            class="numbers"
          >
            <div
              v-for="(value, index) in pulled"
              :key="value"
              class="history-number"
              :class="{
                latest:
                  index === pulled.length - 1,
              }"
            >
              {{ value }}
            </div>
          </div>

          <div
            v-else
            class="empty-history"
          >
            Здесь появятся выпавшие числа
          </div>
        </div>

        <!-- RESET -->

        <div class="bottom">
          <button
            v-if="pulled.length"
            type="button"
            class="reset"
            @click="clear"
          >
            <span>
              ↻
            </span>

            Сбросить
          </button>

          <div class="credits">
            <a
              href="https://www.flaticon.com/ru/free-icons/"
              target="_blank"
              rel="noopener noreferrer"
              title="лото иконки"
            >
              Lotto icon by Magnific — Flaticon
            </a>
          </div>
        </div>
      </section>
    </div>
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
   APP
===================================================== */

.app {
  --accent: #ff3f91;
  --accent-light: #ff78b3;
  --accent-dark: #d41e68;

  --accent-rgb: 255, 63, 145;

  position: relative;

  width: 100%;
  height: 100dvh;

  display: flex;
  align-items: center;
  justify-content: center;

  padding: 14px;

  overflow: hidden;

  color: #ffffff;

  background:
    radial-gradient(
      ellipse at 12% 42%,
      rgba(255, 29, 113, 0.28) 0%,
      rgba(255, 29, 113, 0.09) 28%,
      transparent 52%
    ),
    radial-gradient(
      ellipse at 88% 52%,
      rgba(65, 95, 200, 0.18) 0%,
      transparent 48%
    ),
    linear-gradient(
      108deg,
      #190810 0%,
      #100f17 45%,
      #090c16 100%
    );
}


/* =====================================================
   BOYS THEME
===================================================== */

.app.theme-boys {
  --accent: #748cff;
  --accent-light: #a1b0ff;
  --accent-dark: #4056d8;

  --accent-rgb: 116, 140, 255;

  background:
    radial-gradient(
      ellipse at 12% 42%,
      rgba(74, 88, 175, 0.18),
      transparent 48%
    ),
    radial-gradient(
      ellipse at 88% 52%,
      rgba(85, 110, 255, 0.28),
      transparent 52%
    ),
    linear-gradient(
      108deg,
      #090b14 0%,
      #0e111b 45%,
      #080b15 100%
    );
}


/* =====================================================
   BACKGROUND LIGHTS
===================================================== */

.ambient {
  position: absolute;

  width: 420px;
  height: 420px;

  border-radius: 50%;

  filter: blur(100px);

  pointer-events: none;
}

.ambient-left {
  left: -210px;
  top: 20%;

  background:
    rgba(var(--accent-rgb), 0.2);
}

.ambient-right {
  right: -220px;
  bottom: 10%;

  background:
    rgba(80, 105, 255, 0.13);
}


/* =====================================================
   PEOPLE
===================================================== */

.person {
  position: absolute;

  z-index: 3;

  pointer-events: none;
  user-select: none;
}

.person img {
  display: block;

  width: 100%;
  height: auto;

  filter:
    drop-shadow(
      0 25px 45px
      rgba(0, 0, 0, 0.55)
    );
}


/* Настя слева */

.person-nastya {
  left: 50%;
  top: 50%;

  width: clamp(
    500px,
    42vw,
    680px
  );

  transform:
    translate(-108%, -38%);
}

/* Ваня справа */

.person-vanya {
  left: 50%;
  top: 50%;

  width:
    clamp(
      360px,
      30vw,
      500px
    );

  transform:
    translate(34%, -53%);
}


/* =====================================================
   DECORATIONS
===================================================== */

.party-decoration {
  position: absolute;

  z-index: 1;

  color:
    rgba(var(--accent-rgb), 0.72);

  line-height: 1;

  pointer-events: none;
  user-select: none;

  text-shadow:
    0 0 24px
    rgba(var(--accent-rgb), 0.42);
}

.decor-heart {
  top: 7%;
  left: 6%;

  font-size: 42px;

  transform:
    rotate(-14deg);
}

.decor-sparkle {
  top: 37%;
  left: 4%;

  font-size: 42px;
}

.decor-heart-outline {
  left: 7%;
  bottom: 7%;

  font-size: 55px;

  transform:
    rotate(8deg);
}

.decor-spade {
  top: 7%;
  right: 7%;

  font-size: 43px;

  transform:
    rotate(9deg);
}

.decor-crown {
  top: 41%;
  right: 5%;

  font-size: 54px;

  transform:
    rotate(-4deg);
}

.decor-club {
  right: 7%;
  bottom: 8%;

  font-size: 51px;

  transform:
    rotate(7deg);
}


/* =====================================================
   TEAM TEXT
===================================================== */

.party-text {
  position: absolute;

  z-index: 1;

  color:
    rgba(255, 255, 255, 0.085);

  font-size:
    clamp(
      50px,
      5vw,
      84px
    );

  font-weight: 900;

  line-height: 0.82;

  letter-spacing: -0.055em;

  pointer-events: none;
  user-select: none;
}

.party-text-left {
  left: 2.5%;
  bottom: 17%;

  transform:
    rotate(-7deg);
}

.party-text-right {
  right: 3%;
  top: 17%;

  text-align: right;

  transform:
    rotate(7deg);
}


/* =====================================================
   CONTENT
===================================================== */

.content {
  position: relative;

  z-index: 10;

  width: 100%;
  max-width: 520px;

  display: flex;
  flex-direction: column;

  gap: 9px;
}


/* =====================================================
   PARTY SWITCH
===================================================== */

.party-switch {
  position: relative;

  z-index: 20;

  width: 390px;
  max-width: 100%;

  margin: 0 auto;

  display: grid;
  grid-template-columns: 1fr 1fr;

  padding: 4px;

  border:
    1px solid
    rgba(255, 255, 255, 0.11);

  border-radius: 17px;

  background:
    rgba(8, 9, 14, 0.94);

  backdrop-filter:
    blur(20px);

  box-shadow:
    0 12px 40px
    rgba(0, 0, 0, 0.4);
}

.party-switch-button {
  height: 45px;

  display: flex;
  align-items: center;
  justify-content: center;

  gap: 7px;

  padding:
    5px 12px;

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
    linear-gradient(
      135deg,
      var(--accent-light),
      var(--accent)
    );

  box-shadow:
    0 6px 20px
    rgba(var(--accent-rgb), 0.35);
}

.switch-icon {
  font-size: 16px;
}


/* =====================================================
   GENERATOR
===================================================== */

.generator {
  position: relative;

  width: 100%;

  padding:
    20px 24px 13px;

  overflow: hidden;

  border:
    1px solid
    rgba(255, 255, 255, 0.11);

  border-radius: 25px;

  background:
    linear-gradient(
      145deg,
      rgba(24, 24, 33, 0.95),
      rgba(7, 8, 13, 0.97)
    );

  backdrop-filter:
    blur(26px);

  box-shadow:
    0 35px 90px
    rgba(0, 0, 0, 0.57),
    inset
    0 1px 0
    rgba(255, 255, 255, 0.065);
}

.generator::before {
  content: "";

  position: absolute;

  top: -190px;
  left: 50%;

  width: 300px;
  height: 300px;

  transform:
    translateX(-50%);

  border-radius: 50%;

  background:
    rgba(var(--accent-rgb), 0.22);

  filter:
    blur(65px);

  pointer-events: none;
}


/* =====================================================
   HEADER
===================================================== */

.header {
  position: relative;

  text-align: center;
}

.logo-wrapper {
  width: 42px;
  height: 42px;

  display: flex;
  align-items: center;
  justify-content: center;

  margin:
    0 auto 5px;

  border:
    1px solid
    rgba(var(--accent-rgb), 0.3);

  border-radius: 13px;

  background:
    rgba(var(--accent-rgb), 0.12);
}

.logo {
  display: block;

  width: 27px;
  height: 27px;

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
  margin:
    4px 0 1px;

  font-size: 29px;
  line-height: 1.05;
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

  padding:
    5px 8px;

  border:
    1px solid
    rgba(255, 255, 255, 0.12);

  border-radius: 10px;

  outline: none;

  color: #ffffff;

  background:
    rgba(255, 255, 255, 0.05);

  text-align: center;

  font-size: 15px;
  font-weight: 700;

  transition:
    0.2s ease;
}

.range input:focus {
  border-color:
    rgba(var(--accent-rgb), 0.7);

  box-shadow:
    0 0 0 3px
    rgba(var(--accent-rgb), 0.09);
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
    radial-gradient(
      circle at 35% 25%,
      var(--accent-light),
      var(--accent) 52%,
      var(--accent-dark)
    );

  border:
    1px solid
    rgba(255, 255, 255, 0.24);

  font-size: 47px;
  font-weight: 900;

  line-height: 1;

  box-shadow:
    0 0 40px
    rgba(var(--accent-rgb), 0.35),
    inset
    0 2px 7px
    rgba(255, 255, 255, 0.22);

  transition:
    0.25s ease;
}

.number::after {
  content: "";

  position: absolute;

  inset: 8px;

  border:
    1px solid
    rgba(255, 255, 255, 0.12);

  border-radius: inherit;
}

.number.empty {
  color:
    rgba(255, 255, 255, 0.22);

  background:
    rgba(255, 255, 255, 0.025);

  border:
    1px dashed
    rgba(255, 255, 255, 0.13);

  box-shadow: none;
}

.result-label {
  margin-top: 5px;

  color:
    rgba(255, 255, 255, 0.4);

  font-size: 9px;
}


/* =====================================================
   GENERATE BUTTON
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
    linear-gradient(
      105deg,
      var(--accent-light),
      var(--accent),
      var(--accent-dark)
    );

  font-size: 13px;
  font-weight: 800;

  box-shadow:
    0 10px 28px
    rgba(var(--accent-rgb), 0.26);

  transition:
    0.2s ease;
}

.generate:hover {
  transform:
    translateY(-1px);

  box-shadow:
    0 13px 35px
    rgba(var(--accent-rgb), 0.37);
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
    1px solid
    rgba(var(--accent-rgb), 0.27);

  border-radius: 11px;

  color:
    var(--accent-light);

  background:
    rgba(var(--accent-rgb), 0.075);

  font-size: 13px;
  font-weight: 700;
}


/* =====================================================
   HISTORY
===================================================== */

.history {
  margin-top: 10px;

  padding-top: 8px;

  border-top:
    1px solid
    rgba(255, 255, 255, 0.075);
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
   SMALL HISTORY CIRCLES
===================================================== */

.numbers {
  display: flex;
  flex-wrap: wrap;

  align-items: center;

  gap: 6px;
}

.history-number {
  flex:
    0 0 auto;

  width: 34px;
  height: 34px;

  display: flex;
  align-items: center;
  justify-content: center;

  border:
    1px solid
    rgba(255, 255, 255, 0.09);

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
    0 0 16px
    rgba(var(--accent-rgb), 0.42);
}

.empty-history {
  padding: 8px;

  border:
    1px dashed
    rgba(255, 255, 255, 0.08);

  border-radius: 9px;

  color:
    rgba(255, 255, 255, 0.25);

  text-align: center;

  font-size: 9px;
}


/* =====================================================
   BOTTOM
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

  line-height: 1;
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
   LOWER DESKTOP HEIGHT
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
    width: 34px;
    height: 34px;
  }

  .logo {
    width: 21px;
    height: 21px;
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

  .history-number {
    width: 29px;
    height: 29px;

    font-size: 8px;
  }

  .numbers {
    gap: 4px;
  }

  .reset {
    margin-top: 3px;
  }

  .person-nastya {
    bottom: -150px;
  }
}


/* =====================================================
   TABLET
===================================================== */

@media (max-width: 1050px) {
  .person-nastya {
    opacity: 0.58;

    transform:
      translateX(-95%);
  }

  .person-vanya {
    opacity: 0.6;

    transform:
      translate(33%, -55%);
  }

  .party-text {
    opacity: 0.7;
  }
}


/* =====================================================
   MOBILE
===================================================== */

@media (max-width: 700px) {
  .app {
    padding: 6px;
  }

  .content {
    max-width: 430px;

    gap: 5px;
  }


  /* ===============================
     PEOPLE MOBILE
  ================================ */

  .person-nastya {
    left: -80px;
    bottom: -50px;

    width: 390px;

    opacity: 0.14;

    transform: none;
  }

  .person-vanya {
    top: 110px;
    right: -100px;
    left: auto;

    width: 330px;

    opacity: 0.14;

    transform: none;
  }


  /* ===============================
     DECOR MOBILE
  ================================ */

  .party-text {
    display: none;
  }

  .party-decoration {
    opacity: 0.32;
  }

  .decor-heart {
    top: 12px;
    left: 13px;

    font-size: 27px;
  }

  .decor-spade {
    top: 13px;
    right: 13px;

    font-size: 27px;
  }

  .decor-sparkle {
    left: 6px;

    font-size: 26px;
  }

  .decor-crown {
    right: 6px;

    font-size: 30px;
  }

  .decor-heart-outline {
    left: 6px;
    bottom: 6px;

    font-size: 32px;
  }

  .decor-club {
    right: 7px;
    bottom: 7px;

    font-size: 31px;
  }


  /* ===============================
     SWITCH MOBILE
  ================================ */

  .party-switch {
    width: 100%;

    border-radius: 14px;
  }

  .party-switch-button {
    height: 37px;

    padding:
      4px 8px;

    border-radius: 10px;

    font-size: 10px;
  }

  .switch-icon {
    font-size: 13px;
  }


  /* ===============================
     CARD MOBILE
  ================================ */

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


  /* ===============================
     RANGE MOBILE
  ================================ */

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


  /* ===============================
     NUMBER MOBILE
  ================================ */

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


  /* ===============================
     BUTTON MOBILE
  ================================ */

  .generate,
  .finished {
    min-height: 33px;

    border-radius: 8px;

    font-size: 10px;
  }


  /* ===============================
     HISTORY MOBILE
  ================================ */

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


  /* ===============================
     BOTTOM MOBILE
  ================================ */

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


/* =====================================================
   VERY SMALL MOBILE
===================================================== */

@media (max-width: 380px) {
  .app {
    padding: 4px;
  }

  .content {
    gap: 3px;
  }

  .party-switch {
    padding: 3px;
  }

  .party-switch-button {
    height: 32px;

    font-size: 9px;
  }

  .generator {
    padding:
      7px 9px 5px;
  }

  .logo-wrapper {
    width: 27px;
    height: 27px;
  }

  .logo {
    width: 16px;
    height: 16px;
  }

  .header h1 {
    font-size: 17px;
  }

  .header p {
    font-size: 7px;
  }

  .range {
    margin:
      4px 0 3px;
  }

  .range input {
    height: 26px;
  }

  .range-divider {
    height: 26px;
  }

  .number {
    width: 68px;
    height: 68px;

    font-size: 29px;
  }

  .generate,
  .finished {
    min-height: 29px;
  }

  .history-number {
    width: 24px;
    height: 24px;

    font-size: 7px;
  }

  .numbers {
    gap: 2px;
  }
}
</style>