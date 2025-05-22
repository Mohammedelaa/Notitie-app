<template>
  <div>
    <div class="container mt-4">
      <div class="d-flex justify-content-between align-items-center mb-3">
        <h2>Mijn Notities</h2>
      </div>

      <div class="row mb-3">
        <div class="col-md-4">
          <input v-model="search" type="text" class="form-control" placeholder="Zoeken op titel of tag...">
        </div>
        <div class="col-md-4">
          <input v-model="filterDate" type="date" class="form-control" placeholder="Filter op datum">
        </div>
        <div class="col-md-4">
          <select v-model="filterTag" class="form-select">
            <option value="">Alle labels</option>
            <option v-for="tag in allTags" :key="tag" :value="tag">{{ tag }}</option>
          </select>
        </div>
      </div>

      <div class="row">
        <div class="col-md-4" v-for="note in filteredNotes" :key="note.id">
          <div class="card mb-3" :class="darkMode ? 'bg-secondary text-white' : ''">
            <div class="card-body">
              <h5 class="card-title">{{ note.title }}</h5>
              <div class="card-text" v-html="note.content"></div>
              <p class="card-text">
                <span v-if="note.deadline" class="badge bg-warning text-dark">Deadline: {{ formatDate(note.deadline) }}</span>
              </p>
              <div v-if="note.image">
                <img :src="note.image" class="img-fluid rounded mb-2" alt="Notitie afbeelding">
              </div>
              <div class="mb-2">
                <span class="me-1" v-for="tag in note.tags" :key="tag">
                  <i :class="getIcon(tag)"></i>
                  <span class="badge bg-secondary">{{ tag }}</span>
                </span>
              </div>
              <button class="btn btn-outline-secondary btn-sm me-2" @click="toggleArchive(note.id)">
                {{ note.archived ? 'Herstel' : 'Archiveer' }}
              </button>
              <button class="btn btn-outline-danger btn-sm me-2" @click="remove(note.id)">Verwijder</button>
              <button class="btn btn-outline-info btn-sm" @click="restoreVersion(note.id)">Vorige versie</button>
            </div>
          </div>
        </div>
      </div>

      <div class="mt-4">
        <h3>Nieuwe notitie</h3>
        <form @submit.prevent="addNote">
          <input v-model="newNote.title" placeholder="Titel" class="form-control mb-2">
          <QuillEditor v-model:content="newNote.content" contentType="html" class="mb-2" />
          <input v-model="newNote.tagsInput" placeholder="Tags (gescheiden met komma's)" class="form-control mb-2">
          <input type="datetime-local" v-model="newNote.deadline" class="form-control mb-2">
          <input type="file" @change="handleFileUpload" class="form-control mb-2">
          <button type="submit" class="btn btn-success">Toevoegen</button>
        </form>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'

const router = useRouter()
const search = ref(localStorage.getItem('search') || '')
const filterDate = ref(localStorage.getItem('filterDate') || '')
const filterTag = ref(localStorage.getItem('filterTag') || '')
const darkMode = ref(localStorage.getItem('darkMode') === 'true')

const notes = ref(JSON.parse(localStorage.getItem('notes') || '[]'))
const versions = ref(JSON.parse(localStorage.getItem('versions') || '{}'))

const newNote = ref({ title: '', content: '', tagsInput: '', deadline: '', image: null })

const allTags = computed(() => {
  const tags = new Set()
  notes.value.forEach(n => n.tags && n.tags.forEach(t => tags.add(t)))
  return Array.from(tags)
})

const filteredNotes = computed(() => {
  return notes.value.filter(note =>
    !note.archived &&
    (search.value === '' || note.title.toLowerCase().includes(search.value.toLowerCase()) || note.tags.some(tag => tag.toLowerCase().includes(search.value.toLowerCase()))) &&
    (filterTag.value === '' || note.tags.includes(filterTag.value)) &&
    (filterDate.value === '' || (note.deadline && note.deadline.startsWith(filterDate.value)))
  )
})

watch([search, filterDate, filterTag, darkMode, notes, versions], () => {
  localStorage.setItem('search', search.value)
  localStorage.setItem('filterDate', filterDate.value)
  localStorage.setItem('filterTag', filterTag.value)
  localStorage.setItem('darkMode', darkMode.value)
  localStorage.setItem('notes', JSON.stringify(notes.value))
  localStorage.setItem('versions', JSON.stringify(versions.value))
})

function remove(id) {
  notes.value = notes.value.filter(note => note.id !== id)
}

function toggleArchive(id) {
  const note = notes.value.find(n => n.id === id)
  if (note) note.archived = !note.archived
}

function logout() {
  localStorage.removeItem('loggedIn')
  router.push('/login')
}

function getIcon(tag) {
  const icons = {
    persoonlijk: 'bi bi-person',
    werk: 'bi bi-briefcase',
    school: 'bi bi-book',
    idee: 'bi bi-lightbulb',
    urgent: 'bi bi-exclamation-triangle',
  }
  return icons[tag] || 'bi bi-stickies'
}

function handleFileUpload(event) {
  const file = event.target.files[0]
  if (file) {
    const reader = new FileReader()
    reader.onload = e => newNote.value.image = e.target.result
    reader.readAsDataURL(file)
  }
}

function addNote() {
  if (!newNote.value.title || !newNote.value.content) return alert('Titel en inhoud zijn verplicht')
  const note = {
    id: Date.now(),
    title: newNote.value.title,
    content: newNote.value.content,
    tags: newNote.value.tagsInput.split(',').map(t => t.trim()),
    deadline: newNote.value.deadline,
    image: newNote.value.image,
    archived: false
  }
  notes.value.push(note)
  versions.value[note.id] = [JSON.parse(JSON.stringify(note))]
  newNote.value = { title: '', content: '', tagsInput: '', deadline: '', image: null }
}

function formatDate(dt) {
  return new Date(dt).toLocaleString('nl-NL')
}

function restoreVersion(id) {
  if (!versions.value[id] || versions.value[id].length < 2) return alert('Geen vorige versie beschikbaar')
  const prev = versions.value[id].pop()
  const idx = notes.value.findIndex(n => n.id === id)
  if (idx !== -1) notes.value[idx] = JSON.parse(JSON.stringify(prev))
}

onMounted(() => {
  if (localStorage.getItem('loggedIn') !== 'true') {
    router.push('/login')
  }
})
</script>

<style scoped>
@import '@vueup/vue-quill/dist/vue-quill.snow.css';
</style>
