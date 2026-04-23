<script setup>
import { ref, onMounted, reactive } from 'vue'
import axios from 'axios'

// --- CONFIGURACIÓN ---
const API_URL = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"

// --- ESTADOS ---
const conflicts = ref([])
const loading = ref(true)
const isSubmitting = ref(false)
const toast = reactive({ visible: false, message: '', type: 'success' })

// Formulario reactivo
const newConflict = ref({
  name: '',
  region: '',
  status: 'ACTIVE',
  description: '',
  startDate: new Date().toISOString().split('T')[0]
})

// --- FUNCIONES ---

// 1. Mostrar notificaciones (Toast)
const showToast = (msg, type = 'success') => {
  toast.message = msg
  toast.type = type
  toast.visible = true
  setTimeout(() => { toast.visible = false }, 3000)
}

// 2. Cargar datos (FETCH)
const fetchConflicts = async () => {
  loading.value = true
  try {
    // Forzamos al navegador a pedir datos nuevos con el timestamp (?t=...)
    const { data } = await axios.get(`${API_URL}?t=${Date.now()}`)
    conflicts.value = data
    console.log("Datos recibidos:", data)
  } catch (err) {
    console.error("Error al cargar:", err)
    showToast("Error al conectar con el servidor", "error")
  } finally {
    loading.value = false
  }
}

// 3. Crear conflicto (POST)
const createConflict = async () => {
  if (!newConflict.value.name) return
  
  isSubmitting.value = true
  try {
    // A. Enviamos el dato a Railway
    const response = await axios.post(API_URL, newConflict.value)
    console.log("Respuesta de creación:", response.data)

    // B. Limpiamos el formulario inmediatamente
    newConflict.value = {
      name: '',
      region: '',
      status: 'ACTIVE',
      description: '',
      startDate: new Date().toISOString().split('T')[0]
    }

    showToast("¡Conflicto publicado con éxito!")

    // C. PEQUEÑA ESPERA DE SEGURIDAD (Clave para que salga en la lista)
    // Esto da tiempo a que la DB se actualice tras el POST
    await new Promise(resolve => setTimeout(resolve, 500))

    // D. Refrescamos la lista
    await fetchConflicts()

  } catch (err) {
    console.error("Error al guardar:", err)
    showToast("No se pudo guardar el registro", "error")
  } finally {
    isSubmitting.value = false
  }
}

// Lógica de banderas (ISO 3 a ISO 2 aproximado)
const getFlagUrl = (code) => {
  if (!code) return 'https://flagcdn.com/w40/un.png'
  const iso2 = code.substring(0, 2).toLowerCase()
  return `https://flagcdn.com/w40/${iso2}.png`
}

onMounted(fetchConflicts)
</script>

<template>
  <div class="app-container">
    <Transition name="toast">
      <div v-if="toast.visible" :class="['toast', toast.type]">
        {{ toast.message }}
      </div>
    </Transition>

    <header class="main-header">
      <h1>🌍 Conflict Monitor <span class="live-dot"></span></h1>
    </header>

    <main class="content-grid">
      <aside class="sidebar">
        <div class="glass-card">
          <h2>Registrar Incidente</h2>
          <form @submit.prevent="createConflict" class="modern-form">
            <div class="field">
              <label>Nombre del Conflicto</label>
              <input v-model="newConflict.name" placeholder="Ej: Mar de China Meridional" required />
            </div>
            
            <div class="row">
              <div class="field">
                <label>Región</label>
                <input v-model="newConflict.region" placeholder="Asia" />
              </div>
              <div class="field">
                <label>Estado</label>
                <select v-model="newConflict.status">
                  <option value="ACTIVE">🔴 Activo</option>
                  <option value="TENSION">🟡 Tensión</option>
                  <option value="RESOLVED">🟢 Resuelto</option>
                </select>
              </div>
            </div>

            <div class="field">
              <label>Fecha de Inicio</label>
              <input v-model="newConflict.startDate" type="date" />
            </div>

            <div class="field">
              <label>Descripción</label>
              <textarea v-model="newConflict.description" placeholder="Resumen de la situación..."></textarea>
            </div>

            <button type="submit" class="submit-btn" :disabled="isSubmitting">
              {{ isSubmitting ? 'Publicando...' : 'Publicar Incidente' }}
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
            <p class="desc">{{ c.description }}</p>
            <div class="card-footer">
              <div class="meta">
                <span>📍 {{ c.region || 'Global' }}</span>
                <span>📅 {{ c.startDate }}</span>
              </div>
              <div v-if="c.countries" class="mini-flags">
                 <img v-for="country in c.countries" :key="country.id" :src="getFlagUrl(country.code)" class="tiny-flag" :title="country.name" />
              </div>
            </div>
          </div>
        </div>
      </section>
    </main>
  </div>
</template>

<style scoped>
/* Aquí puedes mantener todo el CSS de tu PDF, 
   solo asegúrate de que estas clases existan o cámbialas por las tuyas */
.app-container {
  background: #0b0f1a;
  color: #f8fafc;
  min-height: 100vh;
  padding: 2rem;
  font-family: 'Inter', sans-serif;
}

.content-grid {
  display: grid;
  grid-template-columns: 380px 1fr;
  gap: 2rem;
  max-width: 1300px;
  margin: 0 auto;
}

.glass-card {
  background: rgba(30, 41, 59, 0.7);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 1.5rem;
  border-radius: 1rem;
  position: sticky;
  top: 2rem;
}

.modern-form { display: flex; flex-direction: column; gap: 1rem; }
.field { display: flex; flex-direction: column; gap: 0.4rem; }
.row { display: grid; grid-template-columns: 1fr 1fr; gap: 1rem; }

input, select, textarea {
  background: #0f172a;
  border: 1px solid #334155;
  color: white;
  padding: 0.7rem;
  border-radius: 0.5rem;
}

.submit-btn {
  background: #10b981;
  color: #064e3b;
  border: none;
  padding: 1rem;
  border-radius: 0.5rem;
  font-weight: bold;
  cursor: pointer;
  transition: 0.2s;
}

.submit-btn:hover:not(:disabled) { background: #34d399; transform: translateY(-2px); }

.conflict-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); gap: 1.5rem; }

.conflict-card {
  background: #1e293b;
  border-radius: 1rem;
  padding: 1.5rem;
  border-bottom: 3px solid #334155;
}

.status-tag { font-size: 0.7rem; font-weight: bold; padding: 0.2rem 0.5rem; border-radius: 0.3rem; }
.active { background: #ef4444; }
.tension { background: #f59e0b; }
.resolved { background: #10b981; }

.live-dot {
  display: inline-block;
  width: 10px; height: 10px;
  background: #10b981;
  border-radius: 50%;
  margin-left: 10px;
  box-shadow: 0 0 10px #10b981;
  animation: pulse 2s infinite;
}

@keyframes pulse {
  0% { transform: scale(0.95); opacity: 0.5; }
  70% { transform: scale(1); opacity: 1; }
  100% { transform: scale(0.95); opacity: 0.5; }
}

.toast {
  position: fixed; top: 20px; right: 20px;
  padding: 1rem 2rem; border-radius: 0.5rem;
  z-index: 1000; color: white; background: #10b981;
}
.toast.error { background: #ef4444; }
</style>