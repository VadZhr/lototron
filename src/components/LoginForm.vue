<script setup>
import { ref } from 'vue';

const emit = defineEmits(['success']);

const AUTH_LOGIN = 'МАЛЬЧИШНИКИДЕВИЧНИК';
const AUTH_PASSWORD = 'ВаНяиНастяСамыеЛучшие1234';

const login = ref('');
const password = ref('');
const error = ref('');

const authorize = () => {
  error.value = '';

  if (
    login.value === AUTH_LOGIN &&
    password.value === AUTH_PASSWORD
  ) {
    sessionStorage.setItem(
      'muzloto-authorized',
      'true'
    );

    emit('success');

    return;
  }

  error.value = 'Неверный логин или пароль';
};
</script>

<template>
  <div class="login-screen">
    <form
      class="login-card"
      @submit.prevent="authorize"
    >
      <div class="login-icon">
        ♫
      </div>

      <span class="login-eyebrow">
        PRIVATE PARTY
      </span>

      <h1>Музлото</h1>

      <p>
        Вход только для своих
      </p>

      <div class="login-fields">
        <label>
          <span>Логин</span>

          <input
            v-model="login"
            type="text"
            autocomplete="username"
            placeholder="Введите логин"
          >
        </label>

        <label>
          <span>Пароль</span>

          <input
            v-model="password"
            type="password"
            autocomplete="current-password"
            placeholder="Введите пароль"
          >
        </label>
      </div>

      <div
        v-if="error"
        class="login-error"
      >
        {{ error }}
      </div>

      <button
        type="submit"
        class="login-button"
      >
        Войти
      </button>
    </form>
  </div>
</template>

<style scoped>
.login-screen {
  position: fixed;
  inset: 0;
  z-index: 2000;

  display: flex;
  align-items: center;
  justify-content: center;

  padding: 20px;

  background: rgba(4, 4, 8, 0.68);

  backdrop-filter: blur(14px);
}

.login-card {
  width: min(430px, 92vw);

  padding: 38px 34px;

  border: 1px solid rgba(255, 255, 255, 0.13);
  border-radius: 28px;

  color: #fff;

  text-align: center;

  background:
    linear-gradient(
      145deg,
      rgba(25, 25, 35, 0.97),
      rgba(7, 8, 13, 0.98)
    );

  box-shadow:
    0 35px 100px rgba(0, 0, 0, 0.65),
    0 0 70px rgba(255, 63, 145, 0.12);
}

.login-icon {
  width: 62px;
  height: 62px;

  display: flex;
  align-items: center;
  justify-content: center;

  margin: 0 auto 12px;

  border-radius: 18px;

  background:
    linear-gradient(
      135deg,
      #ff78b3,
      #ff3f91
    );

  font-size: 30px;

  box-shadow:
    0 10px 30px rgba(255, 63, 145, 0.32);
}

.login-eyebrow {
  display: block;

  color: #ff78b3;

  font-size: 9px;
  font-weight: 900;

  letter-spacing: 0.22em;
}

.login-card h1 {
  margin: 7px 0 3px;

  font-size: 36px;
}

.login-card > p {
  margin: 0;

  color: rgba(255, 255, 255, 0.45);

  font-size: 12px;
}

.login-fields {
  display: flex;
  flex-direction: column;

  gap: 12px;

  margin-top: 28px;

  text-align: left;
}

.login-fields label {
  display: flex;
  flex-direction: column;

  gap: 6px;
}

.login-fields label span {
  color: rgba(255, 255, 255, 0.55);

  font-size: 10px;
  font-weight: 700;
}

.login-fields input {
  width: 100%;
  height: 46px;

  padding: 0 14px;

  border: 1px solid rgba(255, 255, 255, 0.12);
  border-radius: 12px;

  outline: none;

  color: #fff;

  background: rgba(255, 255, 255, 0.05);

  font-size: 14px;
}

.login-fields input:focus {
  border-color: rgba(255, 63, 145, 0.7);

  box-shadow:
    0 0 0 3px rgba(255, 63, 145, 0.09);
}

.login-error {
  margin-top: 12px;

  color: #ff8080;

  font-size: 11px;
  font-weight: 700;
}

.login-button {
  width: 100%;
  height: 46px;

  margin-top: 18px;

  border: none;
  border-radius: 12px;

  color: #fff;

  background:
    linear-gradient(
      105deg,
      #ff78b3,
      #ff3f91,
      #d41e68
    );

  font-size: 13px;
  font-weight: 800;

  cursor: pointer;

  box-shadow:
    0 10px 30px rgba(255, 63, 145, 0.28);

  transition:
    transform 0.2s ease,
    box-shadow 0.2s ease;
}

.login-button:hover {
  transform: translateY(-1px);

  box-shadow:
    0 13px 35px rgba(255, 63, 145, 0.38);
}

.login-button:active {
  transform: scale(0.985);
}
</style>