<script setup>
import { computed, onMounted, ref, watch } from 'vue';

const STORAGE_KEY = 'random-number-generator';

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

const saveToLocalStorage = () => {
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify({
      diapason: diapason.value,
      pulled: pulled.value,
      lastPulled: lastPulled.value,
    })
  );
};

const loadFromLocalStorage = () => {
  const savedData = localStorage.getItem(STORAGE_KEY);

  if (!savedData) return;

  try {
    const parsedData = JSON.parse(savedData);

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
    console.error('Не удалось загрузить данные из localStorage', error);
  }
};

const generateRandom = () => {
  if (isFinished.value) return;

  const { min, max } = diapason.value;

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
    'Вы уверены, что хотите сбросить все выпавшие числа?'
  );

  if (!confirmed) return;

  pulled.value = [];
  lastPulled.value = null;
};

onMounted(() => {
  loadFromLocalStorage();
});

watch(
  [diapason, pulled, lastPulled],
  () => {
    saveToLocalStorage();
  },
  {
    deep: true,
  }
);
</script>

<template>
  <main class="app">
    <section class="generator">
      <header class="header">
        <span class="eyebrow">RANDOM NUMBER</span>
        <h1>Генератор чисел</h1>
        <p>Случайные числа без повторений</p>
      </header>

      <div class="range">
        <label>
          <span>От</span>
          <input
            v-model.number="diapason.min"
            type="number"
          >
        </label>

        <span class="range-divider">—</span>

        <label>
          <span>До</span>
          <input
            v-model.number="diapason.max"
            type="number"
          >
        </label>
      </div>

      <div class="result">
        <div
          class="number"
          :class="{ empty: lastPulled === null }"
        >
          {{ lastPulled ?? '?' }}
        </div>

        <span>
          {{ lastPulled === null ? 'Нажмите кнопку' : 'Последнее число' }}
        </span>
      </div>

      <button
        v-if="!isFinished"
        class="generate"
        @click="generateRandom"
      >
        Сгенерировать число
      </button>

      <div class="history">
        <div class="history-header">
          <h2>Выпавшие числа</h2>

          <span>
            {{ pulled.length }} / {{ totalNumbers }}
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
            :class="{ latest: index === pulled.length - 1 }"
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

      <button
        v-if="pulled.length"
        class="reset"
        @click="clear"
      >
        Сбросить
      </button>
    </section>
  </main>
</template>

<style scoped>
* {
  box-sizing: border-box;
}

.app {
  min-height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;
  padding: 32px 20px;

  background:
    radial-gradient(circle at 50% 20%, #252a42 0%, transparent 35%),
    #10121a;

  color: #f5f6fa;
  font-family: Inter, system-ui, sans-serif;
}

.generator {
  width: 100%;
  max-width: 620px;
  padding: 40px;

  background: rgba(28, 31, 44, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 28px;

  box-shadow: 0 30px 80px rgba(0, 0, 0, 0.35);
}

.header {
  text-align: center;
}

.eyebrow {
  color: #8b8fa3;
  font-size: 12px;
  font-weight: 700;
  letter-spacing: 0.16em;
}

.header h1 {
  margin: 8px 0 4px;
  font-size: 32px;
}

.header p {
  margin: 0;
  color: #9296a8;
}

.range {
  display: flex;
  justify-content: center;
  align-items: end;
  gap: 12px;

  margin: 32px 0;
}

.range label {
  display: flex;
  flex-direction: column;
  gap: 7px;

  color: #9296a8;
  font-size: 13px;
}

.range input {
  width: 90px;
  padding: 11px;

  color: white;
  font-size: 16px;
  text-align: center;

  background: #12141d;
  border: 1px solid #343849;
  border-radius: 10px;
  outline: none;
}

.range input:focus {
  border-color: #8b7cff;
}

.range-divider {
  padding-bottom: 11px;
  color: #65697a;
}

.result {
  display: flex;
  flex-direction: column;
  align-items: center;
  margin: 20px 0 30px;
}

.number {
  display: flex;
  align-items: center;
  justify-content: center;

  width: 170px;
  height: 170px;

  border-radius: 50%;

  background: linear-gradient(145deg, #7967ff, #a178ff);

  font-size: 72px;
  font-weight: 800;

  box-shadow:
    0 15px 50px rgba(121, 103, 255, 0.35),
    inset 0 1px 1px rgba(255, 255, 255, 0.3);
}

.number.empty {
  color: #777b91;
  background: #151822;
  box-shadow: none;
  border: 1px dashed #383c4c;
}

.result span {
  margin-top: 14px;
  color: #85899b;
  font-size: 13px;
}

button {
  font: inherit;
  cursor: pointer;
}

.generate {
  width: 100%;
  padding: 16px;

  color: white;
  font-weight: 700;
  font-size: 16px;

  background: #7464f5;
  border: none;
  border-radius: 13px;

  transition: 0.2s;
}

.generate:hover {
  transform: translateY(-2px);
  background: #8172ff;
  box-shadow: 0 10px 30px rgba(116, 100, 245, 0.25);
}

.generate:active {
  transform: translateY(0);
}

.history {
  margin-top: 32px;
  padding-top: 24px;
  border-top: 1px solid #303342;
}

.history-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-bottom: 16px;
}

.history-header h2 {
  margin: 0;
  font-size: 15px;
}

.history-header span {
  color: #85899b;
  font-size: 13px;
}

.numbers {
  display: flex;
  flex-wrap: wrap;
  gap: 9px;
}

.history-number {
  display: flex;
  align-items: center;
  justify-content: center;

  width: 42px;
  height: 42px;

  background: #13151e;
  border: 1px solid #303342;
  border-radius: 50%;

  color: #a9adbb;
  font-size: 14px;
  font-weight: 600;
}

.history-number.latest {
  color: white;
  background: #7464f5;
  border-color: #7464f5;
}

.empty-history {
  padding: 25px;
  text-align: center;

  color: #65697a;
  font-size: 14px;

  border: 1px dashed #343746;
  border-radius: 12px;
}

.reset {
  display: block;
  margin: 22px auto 0;

  color: #85899b;
  background: none;
  border: none;

  font-size: 13px;
}

.reset:hover {
  color: #fff;
}

@media (max-width: 600px) {
  .generator {
    padding: 28px 20px;
  }

  .number {
    width: 145px;
    height: 145px;
    font-size: 60px;
  }
}
</style>