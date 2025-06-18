<template>
  <div class="evernote-bg">
    <div class="evernote-app">
      <aside class="evernote-sidebar">
        <div class="sidebar-header">
          <span class="sidebar-logo"></span>
          <span class="sidebar-title">Quick Notes</span>
        </div>
        <div class="sidebar-search mb-3">
          <input v-model="search" type="text" class="sidebar-search-input" placeholder="Search notes...">
        </div>
        <div class="sidebar-list">
          <div
            v-for="note in filteredNotes"
            :key="note.id"
            :class="['sidebar-note', selectedNote && selectedNote.id === note.id ? 'active' : '']"
            @click="selectNote(note)"
          >
            <div class="sidebar-note-date">
              <span class="sidebar-note-day">{{ formatDay(note.deadline) }}</span>
              <span class="sidebar-note-month">{{ formatMonth(note.deadline) }}</span>
            </div>
            <div class="sidebar-note-content">
              <div class="sidebar-note-title">{{ note.title }}</div>
              <div class="sidebar-note-snippet" v-html="note.content"></div>
            </div>
          </div>
        </div>
        <button class="sidebar-add-btn" @click="startNewNote">+</button>
      </aside>

      <main class="evernote-main">
        <div v-if="selectedNote" class="note-detail">
          <button class="back-btn" @click="selectedNote = null">&larr; Back To All Notes</button>
          <h2 class="note-title">{{ selectedNote.title }}</h2>
          <div class="note-meta">
            <span v-if="selectedNote.deadline" class="note-date">{{ formatDate(selectedNote.deadline) }}</span>
            <span v-for="tag in selectedNote.tags" :key="tag" class="note-tag">{{ tag }}</span>
          </div>
          <div class="note-content" v-html="selectedNote.content"></div>
          <div v-if="selectedNote.image" class="note-image">
            <img :src="selectedNote.image" alt="Notitie afbeelding" />
          </div>
          <div class="note-actions">
            <button class="btn-delete" @click="remove(selectedNote.id)">Verwijder</button>
          </div>
        </div>
        <div v-else class="note-form">
          <h3>Nieuwe notitie</h3>
          <form @submit.prevent="addNote">
            <input v-model="newNote.title" placeholder="Titel" class="form-control mb-2">
            <QuillEditor v-model:content="newNote.content" contentType="html" class="mb-2" />
            <input v-model="newNote.tagsInput" placeholder="Tags (gescheiden met komma's)" class="form-control mb-2">
            <input type="datetime-local" v-model="newNote.deadline" class="form-control mb-2">
            <input type="file" @change="handleFileUpload" class="form-control mb-2">
            <button type="submit" class="btn btn-success w-100">Toevoegen</button>
          </form>
        </div>
      </main>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue'
import { useRouter } from 'vue-router'
import { QuillEditor } from '@vueup/vue-quill'
import '@vueup/vue-quill/dist/vue-quill.snow.css'

const router = useRouter()
const currentUser = ref(localStorage.getItem('currentUser') || '')

function getNotesKey() {
  return `notes_${currentUser.value}`
}
function getVersionsKey() {
  return `versions_${currentUser.value}`
}

const search = ref(localStorage.getItem('search') || '')
const filterDate = ref(localStorage.getItem('filterDate') || '')
const filterTag = ref(localStorage.getItem('filterTag') || '')
const darkMode = ref(localStorage.getItem('darkMode') === 'true')

const notes = ref(JSON.parse(localStorage.getItem(getNotesKey()) || '[]'))
const versions = ref(JSON.parse(localStorage.getItem(getVersionsKey()) || '{}'))

const newNote = ref({ title: '', content: '', tagsInput: '', deadline: '', image: null })
const selectedNote = ref(null)

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

watch([search, filterDate, filterTag, darkMode, notes, versions, currentUser], () => {
  localStorage.setItem('search', search.value)
  localStorage.setItem('filterDate', filterDate.value)
  localStorage.setItem('filterTag', filterTag.value)
  localStorage.setItem('darkMode', darkMode.value)
  localStorage.setItem(getNotesKey(), JSON.stringify(notes.value))
  localStorage.setItem(getVersionsKey(), JSON.stringify(versions.value))
})

function remove(id) {
  notes.value = notes.value.filter(note => note.id !== id)
  if (selectedNote.value && selectedNote.value.id === id) selectedNote.value = null
}

function toggleArchive(id) {
  const note = notes.value.find(n => n.id === id)
  if (note) note.archived = !note.archived
}

function logout() {
  localStorage.removeItem('loggedIn')
  localStorage.removeItem('currentUser')
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

function selectNote(note) {
  selectedNote.value = note
}
function startNewNote() {
  selectedNote.value = null
}

function formatDay(dt) {
  if (!dt) return ''
  return new Date(dt).getDate().toString().padStart(2, '0')
}
function formatMonth(dt) {
  if (!dt) return ''
  return new Date(dt).toLocaleString('en-US', { month: 'short' }).toUpperCase()
}

onMounted(() => {
  if (localStorage.getItem('loggedIn') !== 'true') {
    router.push('/login')
  }
  currentUser.value = localStorage.getItem('currentUser') || ''
  notes.value = JSON.parse(localStorage.getItem(getNotesKey()) || '[]')
  versions.value = JSON.parse(localStorage.getItem(getVersionsKey()) || '{}')
})
</script>

<style scoped>
@import '@vueup/vue-quill/dist/vue-quill.snow.css';

.evernote-bg {
  min-height: 100vh;
  background: linear-gradient(135deg, #f8fafc 0%, #e9f7ef 100%);
  padding: 0;
  margin: 0;
}
.evernote-app {
  display: flex;
  max-width: 1100px;
  margin: 40px auto;
  background: #fff;
  border-radius: 18px;
  box-shadow: 0 8px 32px rgba(60, 120, 80, 0.08);
  overflow: hidden;
  min-height: 700px;
}

.evernote-sidebar {
  width: 320px;
  background: #f6f8fa;
  border-right: 1px solid #e3e8ee;
  display: flex;
  flex-direction: column;
  position: relative;
  padding: 0;
}
.sidebar-header {
  display: flex;
  align-items: center;
  padding: 28px 24px 16px 24px;
}
.sidebar-logo {
  width: 36px;
  height: 36px;
  background: #ffe066;
  border-radius: 10px;
  margin-right: 12px;
  display: inline-block;
}
.sidebar-title {
  font-size: 1.3rem;
  font-weight: 700;
  color: #3d4a3d;
  letter-spacing: 1px;
}
.sidebar-search {
  padding: 0 24px 12px 24px;
}
.sidebar-search-input {
  width: 100%;
  border-radius: 8px;
  border: 1px solid #e3e8ee;
  padding: 8px 12px;
  font-size: 1rem;
  background: #fff;
  outline: none;
  transition: border 0.2s;
}
.sidebar-search-input:focus {
  border: 1.5px solid #2dbe60;
}
.sidebar-list {
  flex: 1;
  overflow-y: auto;
  padding: 0 8px 0 0;
}
.sidebar-note {
  display: flex;
  align-items: flex-start;
  padding: 16px 18px;
  border-radius: 12px;
  margin: 0 8px 8px 8px;
  cursor: pointer;
  background: #fff;
  transition: background 0.15s, box-shadow 0.15s;
  border: 2px solid transparent;
}
.sidebar-note.active, .sidebar-note:hover {
  background: #e9f7ef;
  border: 2px solid #2dbe60;
}
.sidebar-note-date {
  min-width: 44px;
  text-align: center;
  margin-right: 12px;
  margin-top: 2px;
}
.sidebar-note-day {
  display: block;
  font-size: 1.1rem;
  font-weight: 700;
  color: #2dbe60;
}
.sidebar-note-month {
  display: block;
  font-size: 0.8rem;
  color: #b0b8b0;
  letter-spacing: 1px;
}
.sidebar-note-content {
  flex: 1;
  min-width: 0;
}
.sidebar-note-title {
  font-size: 1rem;
  font-weight: 600;
  color: #3d4a3d;
  margin-bottom: 2px;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.sidebar-note-snippet {
  font-size: 0.92rem;
  color: #7a8b7a;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
.sidebar-add-btn {
  position: absolute;
  bottom: 24px;
  right: 24px;
  width: 48px;
  height: 48px;
  background: #2dbe60;
  color: #fff;
  font-size: 2rem;
  border: none;
  border-radius: 50%;
  box-shadow: 0 2px 8px rgba(45,190,96,0.10);
  cursor: pointer;
  transition: background 0.2s;
  display: flex;
  align-items: center;
  justify-content: center;
}
.sidebar-add-btn:hover {
  background: #24994d;
}

.evernote-main {
  flex: 1;
  padding: 40px 48px;
  background: #fff;
  display: flex;
  flex-direction: column;
  min-width: 0;
}
.note-detail, .note-form {
  width: 100%;
  max-width: 600px;
  margin: 0 auto;
  background: #f8fafc;
  border-radius: 16px;
  box-shadow: 0 2px 8px rgba(45,190,96,0.04);
  padding: 32px 32px 24px 32px;
  min-height: 400px;
  position: relative;
}
.back-btn {
  background: none;
  border: none;
  color: #2dbe60;
  font-size: 1rem;
  font-weight: 600;
  margin-bottom: 16px;
  cursor: pointer;
  padding: 0;
}
.note-title {
  font-size: 1.6rem;
  font-weight: 700;
  color: #3d4a3d;
  margin-bottom: 12px;
}
.note-meta {
  margin-bottom: 18px;
}
.note-date {
  background: #e9f7ef;
  color: #2dbe60;
  padding: 3px 10px;
  border-radius: 8px;
  font-size: 0.95rem;
  margin-right: 8px;
}
.note-tag {
  background: #ffe066;
  color: #bfa600;
  padding: 3px 10px;
  border-radius: 8px;
  font-size: 0.95rem;
  margin-right: 8px;
}
.note-content {
  font-size: 1.08rem;
  color: #4a5a4a;
  margin-bottom: 18px;
}
.note-image img {
  max-width: 100%;
  border-radius: 12px;
  margin-bottom: 18px;
}
.note-actions {
  display: flex;
  gap: 12px;
}
.btn-delete {
  background: #fff;
  color: #e74c3c;
  border: 1.5px solid #e74c3c;
  border-radius: 8px;
  padding: 6px 18px;
  font-weight: 600;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
}
.btn-delete:hover {
  background: #e74c3c;
  color: #fff;
}

.form-control, .form-select {
  border-radius: 8px;
  border: 1px solid #e3e8ee;
  font-size: 1rem;
  padding: 10px 12px;
  margin-bottom: 10px;
}
.form-control:focus, .form-select:focus {
  border-color: #2dbe60;
  box-shadow: 0 0 0 0.1rem #2dbe6044;
}

.btn-success {
  background: #2dbe60 !important;
  border-color: #2dbe60 !important;
  font-weight: 600;
  border-radius: 8px;
  font-size: 1.1rem;
  padding: 10px 0;
}
.btn-success:hover {
  background: #24994d !important;
  border-color: #24994d !important;
}

::-webkit-scrollbar-thumb {
  background: #e9f7ef;
  border-radius: 8px;
}
::-webkit-scrollbar {
  width: 8px;
  background: #f8fafc;
}
</style>
