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

const validatePassword = () => {
  if (!form.password) {
    return 'Введите пароль.'
  }

  const missing = []

  if (form.password.length < 8) {
    missing.push('минимум 8 символов')
  }
  if (!/[a-z]/.test(form.password)) {
    missing.push('строчную букву')
  }
  if (!/[A-Z]/.test(form.password)) {
    missing.push('прописную букву')
  }
  if (!/\d/.test(form.password)) {
    missing.push('цифру')
  }
  if (!/[^A-Za-z0-9]/.test(form.password)) {
    missing.push('специальный символ')
  }

  return missing.length ? `Пароль должен содержать: ${missing.join(', ')}.` : null
}

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

  errors.password = validatePassword()
  if (errors.password) {
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
