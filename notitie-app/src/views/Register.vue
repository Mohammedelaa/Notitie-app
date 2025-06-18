<template>
  <div class="container-fluid p-0 min-vh-100 d-flex flex-column justify-content-center align-items-center" :class="darkMode ? 'custom-dark text-white' : 'bg-white text-dark'">
    <div class="w-100" style="max-width: 400px;">
      <header class="text-center mb-4">
        <h1>Blink</h1>
        <p>Note it in a blink.</p>
      </header>
      <form @submit.prevent="register">
        <div class="mb-3">
          <label for="username" class="form-label">Gebruikersnaam</label>
          <input v-model="username" type="text" class="form-control" id="username" required />
        </div>
        <div class="mb-3">
          <label for="password" class="form-label">Wachtwoord</label>
          <input v-model="password" type="password" class="form-control" id="password" required />
        </div>
        <div class="mb-3">
          <label for="confirm" class="form-label">Bevestig wachtwoord</label>
          <input v-model="confirm" type="password" class="form-control" id="confirm" required />
        </div>
        <button type="submit" class="btn btn-success w-100">Registreren</button>
        <div v-if="error" class="alert alert-danger mt-3">{{ error }}</div>
        <div v-if="success" class="alert alert-success mt-3">Registratie gelukt! Je kunt nu inloggen.</div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const username = ref('')
const password = ref('')
const confirm = ref('')
const error = ref('')
const success = ref(false)
const router = useRouter()
const darkMode = ref(localStorage.getItem('darkMode') === 'true')

function toggleDarkMode() {
  darkMode.value = !darkMode.value
  localStorage.setItem('darkMode', darkMode.value)
  document.body.className = darkMode.value ? 'custom-dark text-white' : 'bg-white text-dark'
}

function register() {
  error.value = ''
  success.value = false
  if (!username.value || !password.value || !confirm.value) {
    error.value = 'Vul alle velden in.'
    return
  }
  if (password.value !== confirm.value) {
    error.value = 'Wachtwoorden komen niet overeen.'
    return
  }
  localStorage.setItem('user', JSON.stringify({ username: username.value, password: password.value }))
  success.value = true
  setTimeout(() => router.push('/login'), 1500)
}
</script>

<style scoped>
nav.navbar {
  min-height: 56px;
}
.navbar-beige {
  background-color: #f5e9da !important;
}
.custom-dark {
  background-color: #343a40 !important;
}
.text-white {
  color: #f8f9fa !important;
}
.bg-white {
  background-color: #fff !important;
}
.text-dark {
  color: #212529 !important;
}
input, textarea, select, .form-control, .form-select {
  background-color: inherit !important;
  color: inherit !important;
  border-color: #bbb !important;
}
input::placeholder, textarea::placeholder {
  color: #888 !important;
}
.btn-outline-secondary {
  color: inherit;
}
form {
  background: rgba(255,255,255,0.02);
  border-radius: 12px;
  padding: 24px 18px 18px 18px;
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.04);
}
.custom-dark form {
  background: rgba(52,58,64,0.9);
  box-shadow: 0 2px 12px 0 rgba(0,0,0,0.12);
}
</style>
