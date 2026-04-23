<script setup>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

const API_URL = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"

const conflicts = ref([])
const loading = ref(true)
const isSubmitting = ref(false)
const toast = reactive({ visible: false, message: '', type: 'success' })

const newConflict = ref({
  name: '',
  region: '',
  status: 'ACTIVE',
  description: '',
  startDate: new Date().toISOString().split('T')[0]
})

const fetchConflicts = async () => {
  loading.value = true
  try {
    const response = await axios.get(`${API_URL}?t=${Date.now()}`)
    // Validamos que los datos sean correctos antes de guardarlos
    conflicts.value = Array.isArray(response.data) ? response.data : []
  } catch (err) {
    console.error("Error:", err)
  } finally {
    loading.value = false
  }
}

const createConflict = async () => {
  if (!newConflict.value.name) return
  isSubmitting.value = true
  try {
    await axios.post(API_URL, newConflict.value)
    newConflict.value = {
      name: '', region: '', status: 'ACTIVE',
      description: '', startDate: new Date().toISOString().split('T')[0]
    }
    showToast("¡Publicado!")
    setTimeout(fetchConflicts, 600)
  } catch (err) {
    showToast("Error al publicar", "error")
  } finally {
    isSubmitting.value = false
  }
}

const showToast = (msg, type = 'success') => {
  toast.message = msg; toast.type = type; toast.visible = true
  setTimeout(() => { toast.visible = false }, 3000)
}

onMounted(fetchConflicts)
</script>

<template>
  <div class="app-container">
    <header class="main-header">
      <h1>🌍 Conflict Monitor</h1>
    </header>

    <main class="content-grid">
      <aside class="sidebar">
        <div class="glass-card">
          <h2>Nuevo Registro</h2>
          <form @submit.prevent="createConflict" class="modern-form">
            <input v-model="newConflict.name" placeholder="Título..." required />
            <div class="row">
              <input v-model="newConflict.region" placeholder="Región" />
              <select v-model="newConflict.status">
                <option value="ACTIVE">ACTIVE</option>
                <option value="TENSION">TENSION</option>
                <option value="RESOLVED">RESOLVED</option>
              </select>
            </div>
            <textarea v-model="newConflict.description" placeholder="Detalles..."></textarea>
            <button type="submit" :disabled="isSubmitting" class="submit-btn">
              {{ isSubmitting ? '...' : 'Publicar' }}
            </button>
          </form>
        </div>
      </aside>

      <section class="feed">
        <div v-if="loading" class="msg">Cargando...</div>
        <div v-else class="conflict-list">
          <div v-for="c in conflicts" :key="c.id" class="conflict-card">
            <div class="card-header">
              <span :class="['status-pill', (c.status || 'ACTIVE').toLowerCase()]">
                {{ c.status || 'ACTIVE' }}
              </span>
              <h3>{{ c.name }}</h3>
            </div>
            <p>{{ c.description }}</p>
            <div class="card-footer">
              <span>📍 {{ c.region || 'N/A' }}</span>
              <span>📅 {{ c.startDate }}</span>
            </div>
          </div>
        </div>
      </section>
    </main>

    <div v-if="toast.visible" :class="['toast', toast.type]">{{ toast.message }}</div>
  </div>
</template>

<style scoped>
.app-container { background: #0f172a; color: white; min-height: 100vh; padding: 20px; font-family: sans-serif; }
.content-grid { display: grid; grid-template-columns: 320px 1fr; gap: 20px; max-width: 1200px; margin: 0 auto; }
.glass-card { background: #1e293b; padding: 20px; border-radius: 10px; }
.modern-form { display: flex; flex-direction: column; gap: 10px; }
input, select, textarea { background: #0f172a; border: 1px solid #334155; color: white; padding: 10px; border-radius: 5px; }
.submit-btn { background: #10b981; border: none; padding: 10px; border-radius: 5px; cursor: pointer; font-weight: bold; }
.conflict-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }
.conflict-card { background: #1e293b; padding: 20px; border-radius: 10px; }
.status-pill { font-size: 10px; padding: 2px 8px; border-radius: 10px; font-weight: bold; }
.active { background: #ef4444; } .tension { background: #f59e0b; } .resolved { background: #10b981; }
.toast { position: fixed; bottom: 20px; right: 20px; background: #10b981; padding: 10px 20px; border-radius: 5px; }
.msg { text-align: center; margin-top: 50px; }
</style>