<template>
  <div class="container-fluid p-0 min-vh-100 d-flex flex-column justify-content-center align-items-center" :class="darkMode ? 'custom-dark text-white' : 'bg-white text-dark'">
    <div class="w-100" style="max-width: 400px;">
      <header class="text-center mb-4">
        <h1>Blink</h1>
        <p>Note it in a blink.</p>
      </header>
      <form @submit.prevent="login">
        <div class="mb-3">
          <label for="username" class="form-label">Gebruikersnaam</label>
          <input v-model="username" type="text" class="form-control" id="username" required />
        </div>
        <div class="mb-3">
          <label for="password" class="form-label">Wachtwoord</label>
          <input v-model="password" type="password" class="form-control" id="password" required />
        </div>
        <button type="submit" class="btn btn-primary w-100">Inloggen</button>
        <router-link to="/register" class="btn btn-link w-100">Nog geen account? Registreren</router-link>
        <div v-if="error" class="alert alert-danger mt-3">{{ error }}</div>
      </form>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const username = ref('')
const password = ref('')
const error = ref('')
const router = useRouter()
const darkMode = ref(localStorage.getItem('darkMode') === 'true')

function login() {
  error.value = ''
  const user = JSON.parse(localStorage.getItem('user') || '{}')
  if (username.value === user.username && password.value === user.password) {
    localStorage.setItem('loggedIn', 'true')
    router.push('/notes')
  } else {
    error.value = 'Ongeldige gebruikersnaam of wachtwoord.'
  }
}
</script>

<style scoped>
.custom-dark {
  background-color: #9cbee0 !important;
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
