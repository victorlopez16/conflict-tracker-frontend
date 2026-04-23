<script setup>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

// --- CONFIGURACIÓN (Asegúrate de que esta URL sea exactamente así) ---
const API_URL = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"

// --- ESTADOS ---
const conflicts = ref([])
const loading = ref(true)
const isSubmitting = ref(false)
const toast = reactive({ visible: false, message: '', type: 'success' })

// Formulario
const newConflict = ref({
  name: '',
  region: '',
  status: 'ACTIVE',
  description: '',
  startDate: new Date().toISOString().split('T')[0]
})

// --- FUNCIONES ---

const showToast = (msg, type = 'success') => {
  toast.message = msg
  toast.type = type
  toast.visible = true
  setTimeout(() => { toast.visible = false }, 3000)
}

// CARGAR DATOS (Corregido para evitar el bucle infinito)
const fetchConflicts = async () => {
  loading.value = true
  try {
    // Añadimos un parámetro de tiempo para saltar la caché
    const res = await axios.get(`${API_URL}?t=${Date.now()}`)
    conflicts.value = Array.isArray(res.data) ? res.data : []
    console.log("Datos cargados correctamente");
  } catch (err) {
    console.error("Error de conexión:", err)
    showToast("Error al conectar con Railway (CORS?)", "error")
    // Si falla, dejamos el array vacío para que deje de cargar
    conflicts.value = []
  } finally {
    loading.value = false
  }
}

const createConflict = async () => {
  if (!newConflict.value.name) return
  isSubmitting.value = true
  try {
    await axios.post(API_URL, newConflict.value)
    
    // Reset form
    newConflict.value = {
      name: '', region: '', status: 'ACTIVE',
      description: '', startDate: new Date().toISOString().split('T')[0]
    }
    
    showToast("¡Conflicto guardado!")
    // Esperar un poco antes de refrescar
    setTimeout(fetchConflicts, 500)
  } catch (err) {
    showToast("Error al publicar", "error")
  } finally {
    isSubmitting.value = false
  }
}

onMounted(fetchConflicts)
</script>

<template>
  <div class="app-container">
    <Transition name="toast">
      <div v-if="toast.visible" :class="['toast', toast.type]">{{ toast.message }}</div>
    </Transition>

    <header class="main-header">
      <h1>🌍 Conflict Monitor <span class="live-dot"></span></h1>
    </header>

    <main class="content-grid">
      <aside class="sidebar">
        <div class="glass-card">
          <h2>Registrar Incidente</h2>
          <form @submit.prevent="createConflict" class="modern-form">
            <input v-model="newConflict.name" placeholder="Nombre..." required />
            <div class="row">
              <input v-model="newConflict.region" placeholder="Región" />
              <select v-model="newConflict.status">
                <option value="ACTIVE">Activo</option>
                <option value="TENSION">Tensión</option>
                <option value="RESOLVED">Resuelto</option>
              </select>
            </div>
            <textarea v-model="newConflict.description" placeholder="Descripción..."></textarea>
            <button type="submit" class="submit-btn" :disabled="isSubmitting">
              {{ isSubmitting ? 'Cargando...' : 'Publicar' }}
            </button>
          </form>
        </div>
      </aside>

      <section class="feed">
        <div v-if="loading" class="loader-container">
          <div class="spinner"></div>
          <p>Sincronizando con Railway...</p>
        </div>

        <div v-else class="conflict-list">
          <div v-for="c in conflicts" :key="c.id" class="conflict-card">
            <div class="card-header">
              <span :class="['status-tag', c.status.toLowerCase()]">{{ c.status }}</span>
              <h3>{{ c.name }}</h3>
            </div>
            <p>{{ c.description }}</p>
            <div class="card-footer">
              <span>📍 {{ c.region }}</span>
              <span>📅 {{ c.startDate }}</span>
            </div>
          </div>
          <div v-if="conflicts.length === 0" class="empty-state">
            No se han encontrado conflictos.
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
/* Estilos básicos para que funcione el diseño */
.app-container { background: #0b0f1a; color: white; min-height: 100vh; padding: 20px; font-family: sans-serif; }
.content-grid { display: grid; grid-template-columns: 350px 1fr; gap: 20px; }
.glass-card { background: #1e293b; padding: 20px; border-radius: 12px; }
.modern-form { display: flex; flex-direction: column; gap: 10px; }
input, select, textarea { background: #0f172a; border: 1px solid #334155; color: white; padding: 10px; border-radius: 5px; }
.submit-btn { background: #10b981; border: none; padding: 10px; border-radius: 5px; cursor: pointer; font-weight: bold; }
.conflict-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 15px; }
.conflict-card { background: #1e293b; padding: 15px; border-radius: 10px; border-bottom: 3px solid #334155; }
.status-tag { font-size: 0.7rem; padding: 2px 5px; border-radius: 3px; text-transform: uppercase; }
.active { background: #ef4444; } .tension { background: #f59e0b; } .resolved { background: #10b981; }
.spinner { width: 40px; height: 40px; border: 4px solid #334155; border-top-color: #10b981; border-radius: 50%; animation: spin 1s linear infinite; margin: 0 auto; }
@keyframes spin { to { transform: rotate(360deg); } }
.live-dot { width: 10px; height: 10px; background: #10b981; border-radius: 50%; display: inline-block; margin-left: 5px; }
</style>