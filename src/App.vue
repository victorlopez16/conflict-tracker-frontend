<template>
  <div style="max-width: 800px; margin: 0 auto; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; color: #e0e0e0; background-color: #1a1a1a; min-height: 100vh; padding: 20px;">
    <h1 style="text-align: center; color: #42b883;">🚀 Conflict Tracker</h1>
    
    <div v-if="loading" style="text-align: center; margin-top: 50px;">
      <p>Cargando datos desde Railway...</p>
    </div>
    
    <div v-else-if="conflicts.length === 0" style="text-align: center; margin-top: 50px;">
      <p>No se encontraron conflictos en la base de datos.</p>
    </div>
    
    <div v-else>
      <div v-for="c in conflicts" :key="c.id" style="border: 1px solid #444; border-radius: 8px; padding: 20px; margin-bottom: 20px; background-color: #242424; box-shadow: 0 4px 10px rgba(0,0,0,0.5);">
        <h3 style="margin-top: 0; color: #646cff; border-bottom: 1px solid #444; padding-bottom: 10px;">{{ c.name }}</h3>
        
        <p><strong>Estado:</strong> 
          <span :style="{ color: c.status === 'ACTIVE' ? '#ff4757' : '#2ed573' }">
            {{ c.status }}
          </span>
        </p>
        
        <p v-if="c.region"><strong>Región:</strong> {{ c.region }}</p>
        
        <p style="font-size: 1em; color: #ccc; line-height: 1.5;">{{ c.description }}</p>
        
        <div style="margin-top: 15px; padding-top: 10px; border-top: 1px dashed #444;">
          <small style="color: #888;">📅 Fecha de inicio: {{ c.startDate || 'No registrada' }}</small>
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
    // IMPORTANTE: Hemos añadido /v1/ porque tu Backend lo requiere
    const baseUrl = import.meta.env.VITE_API_URL 
    const response = await axios.get(`${baseUrl}/conflicts`)
    
    conflicts.value = response.data
    console.log("¡Éxito! Datos cargados:", response.data)
  } catch (error) {
    console.error("Error cargando conflictos. Revisa el CORS o la URL:", error)
  } finally {
    loading.value = false
  }
})
</script>

<style>
/* Reset básico para que el fondo oscuro ocupe toda la pantalla */
body {
  margin: 0;
  background-color: #1a1a1a;
}
</style>