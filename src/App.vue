<template>
  <div class="app-container">
    <header>
      <h1>🌍 Conflict Monitor <span class="badge-live">LIVE</span></h1>
      <p class="api-status">API: {{ API_URL }}</p>
    </header>

    <div class="main-layout">
      <section class="form-container">
        <div class="card">
          <h2>Nuevo Incidente</h2>
          <form @submit.prevent="saveConflict">
            <input v-model="form.name" placeholder="Nombre del conflicto" required />
            <input v-model="form.region" placeholder="Región / Continente" />
            
            <div class="row">
              <select v-model="form.status">
                <option value="ACTIVE">🔴 Activo</option>
                <option value="TENSION">🟡 Tensión</option>
                <option value="RESOLVED">🟢 Resuelto</option>
              </select>
              <input v-model="form.startDate" type="date" required />
            </div>

            <textarea v-model="form.description" placeholder="Descripción de los hechos..."></textarea>
            
            <button type="submit" :disabled="isSending" class="btn-save">
              {{ isSending ? 'Guardando...' : 'Publicar Ahora' }}
            </button>
          </form>
        </div>
      </section>

      <section class="list-container">
        <div class="list-header">
          <h2>Registros Actuales</h2>
          <button @click="loadData" class="btn-refresh" title="Refrescar datos">🔄</button>
        </div>

        <div v-if="isLoading" class="msg">Cargando datos de Railway...</div>
        
        <div v-else class="grid">
          <div v-for="item in conflicts" :key="item.id" class="conflict-card">
            <div class="card-top">
              <span :class="['status', item.status.toLowerCase()]">{{ item.status }}</span>
              <h3>{{ item.name }}</h3>
            </div>
            <p>{{ item.description }}</p>
            <div class="card-bottom">
              <span>📍 {{ item.region }}</span>
              <span>📅 {{ item.startDate }}</span>
            </div>
          </div>
          
          <div v-if="conflicts.length === 0" class="msg">No hay conflictos registrados.</div>
        </div>
      </section>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

// URL de tu API en Railway
const API_URL = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"

// ESTADOS
const conflicts = ref([])
const isLoading = ref(true)
const isSending = ref(false)

const form = ref({
  name: '',
  region: '',
  status: 'ACTIVE',
  description: '',
  startDate: new Date().toISOString().split('T')[0]
})

// 1. CARGAR DATOS (CON ANTI-CACHÉ)
const loadData = async () => {
  isLoading.value = true
  try {
    // Añadimos un timestamp (?t=...) para que el navegador no nos de datos viejos
    const res = await axios.get(`${API_URL}?t=${Date.now()}`)
    conflicts.value = res.data
  } catch (err) {
    console.error("Error al leer:", err)
    alert("Error al conectar con Railway. Revisa el CORS.")
  } finally {
    isLoading.value = false
  }
}

// 2. GUARDAR Y REFRESCAR (LA CLAVE DEL ÉXITO)
const saveConflict = async () => {
  if (isSending.value) return
  
  isSending.value = true
  try {
    // Enviamos el POST
    const response = await axios.post(API_URL, form.value)
    console.log("Servidor respondió:", response.data)

    // Si llegamos aquí, el servidor dijo OK. 
    // Limpiamos el formulario
    form.value = {
      name: '',
      region: '',
      status: 'ACTIVE',
      description: '',
      startDate: new Date().toISOString().split('T')[0]
    }

    // OBLIGAMOS a la app a pedir la lista de nuevo para ver el cambio
    await loadData()
    
  } catch (err) {
    console.error("Error al guardar:", err)
    alert("No se pudo guardar. Revisa si el Backend tiene habilitado el @PostMapping.")
  } finally {
    isSending.value = false
  }
}

onMounted(loadData)
</script>

<style scoped>
:target { scroll-margin-top: 20px; }
.app-container {
  font-family: 'Segoe UI', sans-serif;
  background: #0f172a;
  color: #e2e8f0;
  min-height: 100vh;
  padding: 20px;
}

header { text-align: center; margin-bottom: 30px; }
h1 { color: #10b981; margin: 0; }
.badge-live { background: #ef4444; color: white; font-size: 12px; padding: 2px 6px; border-radius: 4px; vertical-align: middle; }
.api-status { font-family: monospace; font-size: 12px; color: #64748b; }

.main-layout {
  display: grid;
  grid-template-columns: 350px 1fr;
  gap: 30px;
  max-width: 1200px;
  margin: 0 auto;
}

.card { background: #1e293b; padding: 20px; border-radius: 12px; border: 1px solid #334155; }
form { display: flex; flex-direction: column; gap: 15px; }

input, select, textarea {
  background: #0f172a;
  border: 1px solid #334155;
  color: white;
  padding: 10px;
  border-radius: 6px;
}

.row { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; }

.btn-save {
  background: #10b981;
  color: #0f172a;
  border: none;
  padding: 12px;
  font-weight: bold;
  border-radius: 6px;
  cursor: pointer;
}

.btn-save:disabled { opacity: 0.5; }

.list-header { display: flex; justify-content: space-between; align-items: center; }
.btn-refresh { background: none; border: none; font-size: 20px; cursor: pointer; }

.grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(280px, 1fr)); gap: 20px; }

.conflict-card {
  background: #1e293b;
  border-left: 4px solid #334155;
  padding: 15px;
  border-radius: 8px;
}

.card-top { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 10px; }
.status { font-size: 10px; font-weight: bold; padding: 3px 6px; border-radius: 4px; text-transform: uppercase; }
.active { background: #ef4444; color: white; }
.tension { background: #f59e0b; color: white; }
.resolved { background: #10b981; color: white; }

.card-bottom { display: flex; justify-content: space-between; font-size: 11px; color: #94a3b8; margin-top: 15px; }

.msg { text-align: center; padding: 40px; color: #64748b; }

@media (max-width: 850px) {
  .main-layout { grid-template-columns: 1fr; }
}
</style>