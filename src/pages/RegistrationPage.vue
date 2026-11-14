<script setup>
import { ref, reactive } from 'vue'
// Если хочешь использовать axios, раскомментируй:
// import axios from 'axios'

const isLoading = ref(false)
const successMessage = ref('')
const generalError = ref(null)

// Данные формы
const form = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

// Ошибки валидации
const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null
})

// Простая валидация
const validate = () => {
  // Сброс ошибок
  Object.keys(errors).forEach((key) => (errors[key] = null))
  let hasErrors = false

  // Имя: не пустое, длина >= 2
  if (!form.name || form.name.trim().length < 2) {
    errors.name = 'Имя должно быть не короче 2 символов.'
    hasErrors = true
  }

  // Email: не пустой, простой формат
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!form.email || !emailRegex.test(form.email)) {
    errors.email = 'Введите корректный email.'
    hasErrors = true
  }

  // Пароль: не пустой, длина >= 6
  if (!form.password || form.password.length < 6) {
    errors.password = 'Пароль должен быть не короче 6 символов.'
    hasErrors = true
  }

  // Подтверждение пароля: совпадает с паролем
  if (form.password !== form.confirmPassword) {
    errors.confirmPassword = 'Пароли не совпадают.'
    hasErrors = true
  } else if (!form.confirmPassword && form.password) {
    // Если пароль заполнен, а подтверждение пустое
    errors.confirmPassword = 'Подтвердите пароль.'
    hasErrors = true
  }

  return !hasErrors
}

const handleSubmit = async (event) => {
  event.preventDefault() // предотвращаем перезагрузку страницы

  successMessage.value = ''
  generalError.value = null

  if (!validate()) {
    return // если есть ошибки валидации — не отправляем
  }

  isLoading.value = true

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
      confirmPassword: form.confirmPassword // на бэкенд обычно не отправляют confirmPassword
    }

    // Вариант 1: fetch (нативно)
    //api чисто для примера, т.е. тут должна проводиться проверка на бэке
    const response = await fetch('https://jsonplaceholder.typicode.com/users', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify(payload)
    })

    if (!response.ok) {
      throw new Error(`Ошибка сервера: ${response.status}`)
    }

    const data = await response.json()
    successMessage.value =
      'Регистрация успешно отправлена (демо). Данные: ' + JSON.stringify(data, null, 2)
  } catch (err) {
    generalError.value = err instanceof Error ? err.message : 'Произошла ошибка при отправке.'
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="form-page">
    <h1>Регистрация</h1>

    <form @submit="handleSubmit" class="registration-form">
      <!-- Имя -->
      <div class="form-group">
        <label for="name">Имя</label>
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
          @blur="validate"
        />
        <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
      </div>

      <!-- Email -->
      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="form-input"
          :class="{ 'input-error': errors.email }"
          @blur="validate"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>

      <!-- Пароль -->
      <div class="form-group">
        <label for="password">Пароль</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.password }"
          @blur="validate"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
      </div>

      <!-- Подтверждение пароля -->
      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.confirmPassword }"
          @blur="validate"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>

      <!-- Общее сообщение об ошибке -->
      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <!-- Кнопка отправки -->
      <button type="submit" class="btn-submit" :disabled="isLoading">
        {{ isLoading ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>

    <!-- Сообщение об успехе -->
    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>

<style scoped>
.form-page {
  max-width: 420px;
  margin: 40px auto;
  padding: 24px;
  background: #fff;
  border: 1px solid #ddd;
  border-radius: 8px;
}

h1 {
  margin: 0 0 20px;
  font-size: 24px;
  text-align: center;
}

.form-group {
  margin-bottom: 16px;
}

label {
  display: block;
  margin-bottom: 5px;
  font-size: 14px;
}

.form-input {
  width: 100%;
  padding: 8px 10px;
  font-size: 14px;
  border: 1px solid #bbb;
  border-radius: 4px;
}

.form-input:focus {
  outline: none;
  border-color: #4a90e2;
}

.input-error {
  border-color: #d93025;
}

.error-message {
  display: block;
  margin-top: 4px;
  color: #d93025;
  font-size: 13px;
}

.general-error {
  margin-bottom: 16px;
  padding: 8px 10px;
  color: #d93025;
  background: #fdecea;
  border-radius: 4px;
  font-size: 14px;
}

.btn-submit {
  width: 100%;
  padding: 10px;
  font-size: 15px;
  color: #fff;
  background: #4a90e2;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.btn-submit:disabled {
  background: #9bbfe8;
  cursor: not-allowed;
}

.success-message {
  margin-top: 16px;
  padding: 10px;
  color: #1e7e34;
  background: #e7f5ea;
  border-radius: 4px;
  font-size: 14px;
  white-space: pre-wrap;
  word-break: break-word;
}
</style>
