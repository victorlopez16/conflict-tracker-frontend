<template>
  <div style="max-width: 800px; margin: 0 auto; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: #e0e0e0; background-color: #1a1a1a; min-height: 100vh; padding: 20px;">
    <h1 style="text-align: center; color: #42b883;">🚀 Conflict Tracker</h1>
    
    <div v-if="loading" style="text-align: center; margin-top: 50px;">
      <p>Conectando con Railway...</p>
    </div>
    
    <div v-else-if="conflicts.length === 0" style="text-align: center; margin-top: 50px;">
      <p>No hay datos. Revisa la consola (F12).</p>
    </div>
    
    <div v-else>
      <div v-for="c in conflicts" :key="c.id" style="border: 1px solid #444; border-radius: 8px; padding: 20px; margin-bottom: 20px; background-color: #242424; box-shadow: 0 4px 10px rgba(0,0,0,0.5);">
        <h3 style="margin-top: 0; color: #646cff; border-bottom: 1px solid #444; padding-bottom: 10px;">
          {{ c.name || 'Sin nombre' }}
        </h3>
        
        <p><strong>Estado:</strong> 
          <span :style="{ color: c.status === 'ACTIVE' ? '#ff4757' : '#2ed573' }">
            {{ c.status || 'UNKNOWN' }}
          </span>
        </p>
        
        <p v-if="c.region"><strong>Región:</strong> {{ c.region }}</p>
        
        <p style="font-size: 1em; color: #ccc; line-height: 1.5;">{{ c.description }}</p>
        
        <div v-if="c.countries && c.countries.length" style="margin: 10px 0;">
          <strong>Países involucrados:</strong>
          <span v-for="country in c.countries" :key="country.id" style="display: inline-block; background: #333; padding: 2px 8px; border-radius: 4px; margin-left: 5px; font-size: 0.8em;">
            {{ country.name }}
          </span>
        </div>

        <div style="margin-top: 15px; padding-top: 10px; border-top: 1px dashed #444;">
          <small style="color: #888;">📅 Inicio: {{ c.startDate || 'No registrada' }}</small>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const conflicts = ref([])
const loading = ref(true)

onMounted(async () => {
  try {
    const url = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"
    
    console.log("Llamando a:", url)
    const response = await axios.get(url)
    
    // Forzamos a que conflicts sea el array de datos
    conflicts.value = Array.isArray(response.data) ? response.data : [response.data]
    
    console.log("Datos cargados correctamente:", conflicts.value)
  } catch (error) {
    console.error("Error detallado:", error)
    if (error.response) {
      console.error("Respuesta del servidor:", error.response.status)
    }
  } finally {
    loading.value = false
  }
})
</script>

<style>
body { margin: 0; background-color: #1a1a1a; }
</style>