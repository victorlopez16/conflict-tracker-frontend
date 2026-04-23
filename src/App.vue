<script setup>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

// 1. URL DE TU API (Verificada)
const API_URL = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"

// 2. ESTADOS REACTIVOS
const conflicts = ref([]) // Aquí se guardan los datos
const loading = ref(true)
const isSubmitting = ref(false)
const toast = reactive({ visible: false, message: '', type: 'success' })

// Formulario para el POST
const newConflict = ref({
  name: '',
  region: '',
  status: 'ACTIVE',
  description: '',
  startDate: new Date().toISOString().split('T')[0]
})

// 3. FUNCIONES DE CARGA Y GUARDADO
const fetchConflicts = async () => {
  loading.value = true
  try {
    // Usamos el "timestamp" ?t= para obligar a Vercel/Navegador a traer datos nuevos
    const response = await axios.get(`${API_URL}?t=${Date.now()}`)
    
    // IMPORTANTE: Nos aseguramos de que lo que guardamos es un Array
    conflicts.value = Array.isArray(response.data) ? response.data : []
    
    console.log("Datos cargados en el cliente:", conflicts.value)
  } catch (err) {
    console.error("Error al obtener conflictos:", err)
    showToast("Error de conexión con el servidor", "error")
  } finally {
    loading.value = false
  }
}

const createConflict = async () => {
  if (!newConflict.value.name) return
  isSubmitting.value = true
  
  try {
    // A. Enviar a Railway
    await axios.post(API_URL, newConflict.value)
    
    // B. Limpiar formulario
    newConflict.value = {
      name: '', region: '', status: 'ACTIVE',
      description: '', startDate: new Date().toISOString().split('T')[0]
    }
    
    showToast("¡Incidente publicado!")

    // C. ESPERA DE SEGURIDAD Y REFRESCO
    // Esperamos 600ms para que a la base de datos le dé tiempo a registrarlo
    setTimeout(async () => {
      await fetchConflicts()
    }, 600)

  } catch (err) {
    console.error("Error al guardar:", err)
    showToast("Error al publicar", "error")
  } finally {
    isSubmitting.value = false
  }
}

const showToast = (msg, type = 'success') => {
  toast.message = msg
  toast.type = type
  toast.visible = true
  setTimeout(() => { toast.visible = false }, 3000)
}

onMounted(fetchConflicts)
</script>

<template>
  <div class="app-container">
    <header class="main-header">
      <h1>🌍 Conflict Monitor <span class="live-indicator">LIVE</span></h1>
    </header>

    <main class="content-grid">
      <aside class="sidebar">
        <div class="glass-card">
          <h2>Nuevo Registro</h2>
          <form @submit.prevent="createConflict" class="modern-form">
            <input v-model="newConflict.name" placeholder="Título del conflicto..." required />
            <div class="row">
              <input v-model="newConflict.region" placeholder="Región" />
              <select v-model="newConflict.status">
                <option value="ACTIVE">🔴 Activo</option>
                <option value="TENSION">🟡 Tensión</option>
                <option value="RESOLVED">🟢 Resuelto</option>
              </select>
            </div>
            <textarea v-model="newConflict.description" placeholder="Detalles..."></textarea>
            <button type="submit" :disabled="isSubmitting" class="submit-btn">
              {{ isSubmitting ? 'Procesando...' : 'Publicar ahora' }}
            </button>
          </form>
        </div>
      </aside>

      <section class="feed">
        <div v-if="loading" class="state-msg">
          <div class="spinner"></div>
          <p>Conectando con Railway...</p>
        </div>

        <div v-else class="conflict-list">
          <div v-for="c in conflicts" :key="c.id" class="conflict-card">
            <div class="card-header">
              <span :class="['status-pill', c.status.toLowerCase()]">{{ c.status }}</span>
              <h3>{{ c.name }}</h3>
            </div>
            <p class="description">{{ c.description }}</p>
            <div class="card-footer">
              <span class="meta-item">📍 {{ c.region || 'N/A' }}</span>
              <span class="meta-item">📅 {{ c.startDate }}</span>
            </div>
          </div>

          <div v-if="conflicts.length === 0" class="state-msg">
            <p>No hay conflictos registrados en la base de datos.</p>
          </div>
        </div>
      </section>
    </main>

    <div v-if="toast.visible" :class="['toast-popup', toast.type]">
      {{ toast.message }}
    </div>
  </div>
</template>

<style scoped>
/* ESTILOS ESENCIALES PARA QUE SE VEA BIEN */
.app-container { background: #0f172a; color: white; min-height: 100vh; padding: 2rem; }
.content-grid { display: grid; grid-template-columns: 350px 1fr; gap: 2rem; max-width: 1200px; margin: 0 auto; }
.glass-card { background: #1e293b; padding: 1.5rem; border-radius: 1rem; border: 1px solid #334155; }
.modern-form { display: flex; flex-direction: column; gap: 1rem; }
input, select, textarea { background: #0f172a; border: 1px solid #334155; color: white; padding: 0.8rem; border-radius: 0.5rem; }
.submit-btn { background: #10b981; color: #064e3b; border: none; padding: 1rem; border-radius: 0.5rem; font-weight: bold; cursor: pointer; }
.submit-btn:disabled { opacity: 0.5; }

.conflict-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 1.5rem; }
.conflict-card { background: #1e293b; padding: 1.5rem; border-radius: 1rem; border-bottom: 4px solid #334155; transition: 0.3s; }
.conflict-card:hover { transform: translateY(-5px); border-color: #10b981; }

.status-pill { font-size: 0.7rem; font-weight: bold; padding: 0.2rem 0.6rem; border-radius: 2rem; text-transform: uppercase; }
.active { background: #ef4444; } .tension { background: #f59e0b; } .resolved { background: #10b981; }

.card-footer { display: flex; justify-content: space-between; margin-top: 1rem; font-size: 0.8rem; color: #94a3b8; }
.spinner { width: 40px; height: 40px; border: 4px solid #334155; border-top-color: #10b981; border-radius: 50%; animation: spin 1s linear infinite; margin-bottom: 1rem; }
@keyframes spin { to { transform: rotate(360deg); } }
.state-msg { display: flex; flex-direction: column; align-items: center; justify-content: center; height: 300px; color: #64748b; }
.toast-popup { position: fixed; bottom: 20px; right: 20px; background: #10b981; padding: 1rem 2rem; border-radius: 0.5rem; box-shadow: 0 10px 15px rgba(0,0,0,0.3); }
</style>