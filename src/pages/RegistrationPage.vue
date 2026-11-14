<script setup>
import { reactive, ref } from 'vue'

const isLoading = ref(false)
const successMessage = ref('')
const generalError = ref(null)

const form = reactive({
  name: '',
  email: '',
  password: '',
  confirmPassword: ''
})

const errors = reactive({
  name: null,
  email: null,
  password: null,
  confirmPassword: null
})

const validate = () => {
  Object.keys(errors).forEach((key) => (errors[key] = null))
  let hasErrors = false

  if (!form.name || form.name.trim().length < 2) {
    errors.name = 'Имя должно быть не короче 2 символов.'
    hasErrors = true
  }

  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!form.email || !emailRegex.test(form.email)) {
    errors.email = 'Введите корректный email.'
    hasErrors = true
  }

  if (!form.password || form.password.length < 6) {
    errors.password = 'Пароль должен быть не короче 6 символов.'
    hasErrors = true
  }

  if (form.password !== form.confirmPassword) {
    errors.confirmPassword = 'Пароли не совпадают.'
    hasErrors = true
  } else if (!form.confirmPassword && form.password) {
    errors.confirmPassword = 'Подтвердите пароль.'
    hasErrors = true
  }

  return !hasErrors
}

const handleSubmit = async (event) => {
  event.preventDefault()

  successMessage.value = ''
  generalError.value = null

  if (!validate()) {
    return
  }

  isLoading.value = true

  try {
    const payload = {
      name: form.name.trim(),
      email: form.email.trim(),
      password: form.password,
      confirmPassword: form.confirmPassword
    }

    const response = await fetch('https://jsonplaceholder.typicode.com/posts', {
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

    <form class="registration-form" @submit="handleSubmit">
      <div class="form-group">
        <label for="name">Имя</label>
        <input
          id="name"
          v-model="form.name"
          type="text"
          class="form-input"
          :class="{ 'input-error': errors.name }"
        />
        <span v-if="errors.name" class="error-message">{{ errors.name }}</span>
      </div>

      <div class="form-group">
        <label for="email">Email</label>
        <input
          id="email"
          v-model="form.email"
          type="email"
          class="form-input"
          :class="{ 'input-error': errors.email }"
        />
        <span v-if="errors.email" class="error-message">{{ errors.email }}</span>
      </div>

      <div class="form-group">
        <label for="password">Пароль</label>
        <input
          id="password"
          v-model="form.password"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.password }"
        />
        <span v-if="errors.password" class="error-message">{{ errors.password }}</span>
      </div>

      <div class="form-group">
        <label for="confirmPassword">Подтверждение пароля</label>
        <input
          id="confirmPassword"
          v-model="form.confirmPassword"
          type="password"
          class="form-input"
          :class="{ 'input-error': errors.confirmPassword }"
        />
        <span v-if="errors.confirmPassword" class="error-message">{{ errors.confirmPassword }}</span>
      </div>

      <div v-if="generalError" class="general-error">{{ generalError }}</div>

      <button type="submit" class="btn-submit" :disabled="isLoading">
        {{ isLoading ? 'Отправка...' : 'Зарегистрироваться' }}
      </button>
    </form>

    <div v-if="successMessage" class="success-message">
      {{ successMessage }}
    </div>
  </div>
</template>
