<template>
  <div :class="['container-fluid p-0 min-vh-100', darkMode ? 'custom-dark text-white' : 'bg-white text-dark']">
    <nav class="navbar navbar-expand border-bottom mb-4 navbar-beige" :class="darkMode ? 'custom-dark navbar-dark border-secondary' : ''">
      <div class="container-fluid justify-content-between align-items-center gap-2">
        <div class="navbar-brand d-flex align-items-center">
          <img src="/logo.png" alt="Logo" height="36" class="me-2" />
        </div>
        <div class="d-flex align-items-center gap-2">
          <button class="btn btn-outline-secondary me-2" @click="toggleDarkMode">
            <i :class="darkMode ? 'bi bi-brightness-high-fill' : 'bi bi-moon-stars-fill'"></i>
            {{ darkMode ? 'Light' : 'Dark' }} Mode
          </button>
          <button class="btn btn-danger" @click="logout">Uitloggen</button>
        </div>
      </div>
    </nav>
    <router-view />
  </div>
</template>

<script setup>
import { ref, watch } from 'vue'
import { useRouter } from 'vue-router'

const darkMode = ref(localStorage.getItem('darkMode') === 'true')
const router = useRouter()

function toggleDarkMode() {
  darkMode.value = !darkMode.value
  localStorage.setItem('darkMode', darkMode.value)
  document.body.className = darkMode.value ? 'custom-dark text-white' : 'bg-white text-dark'
}

function logout() {
  localStorage.removeItem('loggedIn')
  router.push('/login')
}

watch(darkMode, (val) => {
  document.body.className = val ? 'custom-dark text-white' : 'bg-white text-dark'
})
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
</style>
