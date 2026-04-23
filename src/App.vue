<template>
  <div class="app-container">

    <!-- TOAST -->
    <Transition name="toast">
      <div v-if="toast.visible" :class="['toast', toast.type]">
        <span class="toast-icon">{{ toast.type === 'success' ? '✓' : '✗' }}</span>
        {{ toast.message }}
      </div>
    </Transition>

    <!-- HEADER -->
    <header class="main-header">
      <div class="header-left">
        <div class="logo-mark">
          <svg viewBox="0 0 36 36" fill="none" xmlns="http://www.w3.org/2000/svg" class="globe-svg">
            <circle cx="18" cy="18" r="16" stroke="currentColor" stroke-width="1.5"/>
            <ellipse cx="18" cy="18" rx="7" ry="16" stroke="currentColor" stroke-width="1.5"/>
            <line x1="2" y1="18" x2="34" y2="18" stroke="currentColor" stroke-width="1.5"/>
            <line x1="5" y1="10" x2="31" y2="10" stroke="currentColor" stroke-width="1"/>
            <line x1="5" y1="26" x2="31" y2="26" stroke="currentColor" stroke-width="1"/>
          </svg>
        </div>
        <div>
          <h1>Global Conflict Monitor</h1>
          <p class="subtitle">Sistema de seguimiento de incidentes geopolíticos en tiempo real</p>
        </div>
      </div>
      <div class="header-right">
        <div class="live-chip">
          <span class="pulse-dot"></span>
          LIVE
        </div>
        <div class="stats-chips">
          <div class="chip chip--red">
            <span class="chip-value">{{ countByStatus('ACTIVE') }}</span>
            <span class="chip-label">Activos</span>
          </div>
          <div class="chip chip--yellow">
            <span class="chip-value">{{ countByStatus('TENSION') }}</span>
            <span class="chip-label">Tensión</span>
          </div>
          <div class="chip chip--green">
            <span class="chip-value">{{ countByStatus('RESOLVED') }}</span>
            <span class="chip-label">Resueltos</span>
          </div>
        </div>
      </div>
    </header>

    <main class="content">

      <!-- SIDEBAR -->
      <aside class="sidebar">

        <!-- MAPA DE REGIÓN VISUAL -->
        <div class="card map-card">
          <h3 class="card-title">
            <svg viewBox="0 0 20 20" fill="currentColor" class="icon"><path fill-rule="evenodd" d="M5.05 4.05a7 7 0 119.9 9.9L10 18.9l-4.95-4.95a7 7 0 010-9.9zM10 11a2 2 0 100-4 2 2 0 000 4z" clip-rule="evenodd"/></svg>
            Distribución Regional
          </h3>
          <div class="region-bars">
            <div
              v-for="region in regionStats"
              :key="region.name"
              class="region-row"
              :title="`${region.name}: ${region.total} conflicto(s)`"
            >
              <span class="region-name">{{ region.name }}</span>
              <div class="region-track">
                <div
                  class="region-fill"
                  :style="{
                    width: regionBarWidth(region.total) + '%',
                    background: regionColor(region.name)
                  }"
                ></div>
              </div>
              <div class="region-dots">
                <span v-if="region.active > 0" class="rdot rdot--red" :title="`${region.active} activo(s)`">{{ region.active }}</span>
                <span v-if="region.tension > 0" class="rdot rdot--yellow" :title="`${region.tension} en tensión`">{{ region.tension }}</span>
                <span v-if="region.resolved > 0" class="rdot rdot--green" :title="`${region.resolved} resuelto(s)`">{{ region.resolved }}</span>
              </div>
            </div>
            <p v-if="regionStats.length === 0" class="region-empty">Sin datos regionales aún</p>
          </div>
        </div>

        <!-- FILTROS -->
        <div class="card filter-card">
          <h3 class="card-title">
            <svg viewBox="0 0 20 20" fill="currentColor" class="icon"><path fill-rule="evenodd" d="M3 3a1 1 0 011-1h12a1 1 0 011 1v3a1 1 0 01-.293.707L13 10.414V15a1 1 0 01-.553.894l-4 2A1 1 0 017 17v-6.586L3.293 6.707A1 1 0 013 6V3z" clip-rule="evenodd"/></svg>
            Filtros
          </h3>
          <div class="filter-group">
            <label>Buscar</label>
            <div class="search-wrap">
              <svg viewBox="0 0 20 20" fill="currentColor" class="search-icon"><path fill-rule="evenodd" d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z" clip-rule="evenodd"/></svg>
              <input v-model="filters.search" placeholder="Nombre, región..." class="search-input" />
              <button v-if="filters.search" @click="filters.search = ''" class="clear-btn" title="Limpiar">✕</button>
            </div>
          </div>
          <div class="filter-group">
            <label>Estado</label>
            <div class="filter-pills">
              <button
                v-for="s in statusOptions"
                :key="s.value"
                @click="toggleStatusFilter(s.value)"
                :class="['pill', s.cls, { active: filters.statuses.includes(s.value) }]"
              >{{ s.label }}</button>
            </div>
          </div>
          <div class="filter-group">
            <label>Región</label>
            <select v-model="filters.region" class="select-input">
              <option value="">Todas las regiones</option>
              <option v-for="r in uniqueRegions" :key="r" :value="r">{{ r }}</option>
            </select>
          </div>
          <div class="filter-group">
            <label>Ordenar por</label>
            <select v-model="filters.sortBy" class="select-input">
              <option value="date_desc">Fecha (más reciente)</option>
              <option value="date_asc">Fecha (más antiguo)</option>
              <option value="name_asc">Nombre A→Z</option>
              <option value="name_desc">Nombre Z→A</option>
            </select>
          </div>
          <button v-if="hasActiveFilters" @click="resetFilters" class="btn-reset">
            Limpiar filtros
          </button>
        </div>

        <!-- FORMULARIO -->
        <div class="card form-card">
          <h3 class="card-title">
            <svg viewBox="0 0 20 20" fill="currentColor" class="icon"><path fill-rule="evenodd" d="M10 3a1 1 0 011 1v5h5a1 1 0 110 2h-5v5a1 1 0 11-2 0v-5H4a1 1 0 110-2h5V4a1 1 0 011-1z" clip-rule="evenodd"/></svg>
            Registrar Incidente
          </h3>

          <!-- ✅ FIX #4: Se usa @submit.prevent, no se pinta nada hasta recibir ID del servidor -->
          <form @submit.prevent="createConflict" class="conflict-form" novalidate>
            <div class="input-group" :class="{ error: formErrors.name }">
              <label>Nombre del Conflicto <span class="required">*</span></label>
              <input
                v-model="newConflict.name"
                placeholder="Ej: Tensiones en el Mar Rojo"
                @blur="validateField('name')"
                :disabled="creating"
              />
              <span v-if="formErrors.name" class="error-msg">{{ formErrors.name }}</span>
            </div>

            <div class="input-row">
              <div class="input-group">
                <label>Región</label>
                <input v-model="newConflict.region" placeholder="Ej: Medio Oriente" :disabled="creating" />
              </div>
              <div class="input-group">
                <label>Estado</label>
                <select v-model="newConflict.status" :disabled="creating">
                  <option value="ACTIVE">🔴 Activo</option>
                  <option value="TENSION">🟡 Tensión</option>
                  <option value="RESOLVED">🟢 Resuelto</option>
                </select>
              </div>
            </div>

            <div class="input-group" :class="{ error: formErrors.startDate }">
              <label>Fecha de Inicio <span class="required">*</span></label>
              <input
                v-model="newConflict.startDate"
                type="date"
                :max="today"
                @blur="validateField('startDate')"
                :disabled="creating"
              />
              <span v-if="formErrors.startDate" class="error-msg">{{ formErrors.startDate }}</span>
            </div>

            <div class="input-group">
              <label>Descripción</label>
              <textarea
                v-model="newConflict.description"
                rows="3"
                placeholder="Detalles del conflicto..."
                maxlength="500"
                :disabled="creating"
              ></textarea>
              <span class="char-count">{{ newConflict.description.length }}/500</span>
            </div>

            <button type="submit" :disabled="creating || !isFormValid" class="btn-submit">
              <span v-if="creating" class="btn-spinner"></span>
              {{ creating ? 'Guardando...' : 'Publicar Conflicto' }}
            </button>
          </form>
        </div>
      </aside>

      <!-- FEED -->
      <section class="feed">
        <div class="feed-header">
          <p class="result-count">
            <template v-if="!loading">
              <strong>{{ filteredConflicts.length }}</strong> resultado{{ filteredConflicts.length !== 1 ? 's' : '' }}
              <span v-if="hasActiveFilters"> (filtrado de {{ conflicts.length }})</span>
            </template>
          </p>
          <!-- ✅ FIX #3: Botón de refresh llama a fetchConflicts que añade timestamp -->
          <button @click="fetchConflicts" class="btn-refresh" :class="{ spinning: loading }" title="Actualizar" :disabled="loading">
            <svg viewBox="0 0 20 20" fill="currentColor" class="icon"><path fill-rule="evenodd" d="M4 2a1 1 0 011 1v2.101a7.002 7.002 0 0111.601 2.566 1 1 0 11-1.885.666A5.002 5.002 0 005.999 7H9a1 1 0 010 2H4a1 1 0 01-1-1V3a1 1 0 011-1zm.008 9.057a1 1 0 011.276.61A5.002 5.002 0 0014.001 13H11a1 1 0 110-2h5a1 1 0 011 1v5a1 1 0 11-2 0v-2.101a7.002 7.002 0 01-11.601-2.566 1 1 0 01.61-1.276z" clip-rule="evenodd"/></svg>
            Actualizar
          </button>
        </div>

        <!-- LOADING -->
        <div v-if="loading" class="state-msg">
          <div class="skeleton-grid">
            <div v-for="n in 6" :key="n" class="skeleton-card">
              <div class="skeleton-line short"></div>
              <div class="skeleton-line"></div>
              <div class="skeleton-line medium"></div>
            </div>
          </div>
        </div>

        <!-- ERROR -->
        <div v-else-if="loadError" class="state-msg state-error">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" class="state-icon"><circle cx="12" cy="12" r="10"/><line x1="12" y1="8" x2="12" y2="12"/><circle cx="12" cy="16" r="1" fill="currentColor"/></svg>
          <p>No se pudo conectar con el servidor.</p>
          <button @click="fetchConflicts" class="btn-retry">Reintentar</button>
        </div>

        <!-- VACÍO -->
        <div v-else-if="filteredConflicts.length === 0" class="state-msg">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" class="state-icon"><path d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"/></svg>
          <p>No se encontraron conflictos con los filtros actuales.</p>
          <button v-if="hasActiveFilters" @click="resetFilters" class="btn-retry">Quitar filtros</button>
        </div>

        <!-- GRID DE CONFLICTOS -->
        <!-- ✅ FIX #4: La key usa c.id que solo existe una vez el backend confirmó el objeto -->
        <TransitionGroup v-else name="card-list" tag="div" class="grid-layout">
          <div
            v-for="c in filteredConflicts"
            :key="c.id"
            class="card conflict-card"
            :class="`status-${c.status.toLowerCase()}`"
            @click="openDetail(c)"
            role="button"
            tabindex="0"
            @keydown.enter="openDetail(c)"
          >
            <div class="card-top">
              <div class="card-header">
                <span :class="['badge', c.status.toLowerCase()]">
                  {{ statusLabel(c.status) }}
                </span>
                <span class="card-age">{{ timeAgo(c.startDate) }}</span>
              </div>
              <h3 class="card-title-conflict">{{ c.name }}</h3>
              <p class="description">{{ c.description || 'Sin descripción.' }}</p>
            </div>

            <div class="card-footer">
              <div class="meta">
                <span class="meta-item">
                  <svg viewBox="0 0 16 16" fill="currentColor" class="meta-icon"><path fill-rule="evenodd" d="M8 1.5a6.5 6.5 0 100 13 6.5 6.5 0 000-13zM0 8a8 8 0 1116 0A8 8 0 010 8zm8-3.5a.5.5 0 01.5.5v3.793l2.146 2.147a.5.5 0 01-.707.707l-2.293-2.293A.5.5 0 017.5 9V5a.5.5 0 01.5-.5z"/></svg>
                  {{ formatDate(c.startDate) }}
                </span>
                <span v-if="c.region" class="meta-item">
                  <svg viewBox="0 0 16 16" fill="currentColor" class="meta-icon"><path fill-rule="evenodd" d="M8 1a3 3 0 100 6 3 3 0 000-6zM4 4a4 4 0 118 0 4 4 0 01-8 0zm-1.5 7a.5.5 0 00-.5.5v.5c0 1.1.9 2 2 2h7a2 2 0 002-2v-.5a.5.5 0 00-.5-.5h-10z"/></svg>
                  {{ c.region }}
                </span>
              </div>

              <div v-if="c.countries && c.countries.length" class="flags-row">
                <div
                  v-for="country in c.countries.slice(0, 5)"
                  :key="country.id"
                  class="flag-chip"
                  :title="country.name"
                >
                  <img
                    :src="getFlagUrl(country.code)"
                    :alt="country.name"
                    class="flag-img"
                    loading="lazy"
                    @error="handleFlagError"
                  />
                  <span>{{ country.code }}</span>
                </div>
                <span v-if="c.countries.length > 5" class="flag-overflow">
                  +{{ c.countries.length - 5 }}
                </span>
              </div>
            </div>
          </div>
        </TransitionGroup>
      </section>
    </main>

    <!-- MODAL DETALLE -->
    <Transition name="modal">
      <div v-if="selectedConflict" class="modal-overlay" @click.self="closeDetail">
        <div class="modal" role="dialog" aria-modal="true">
          <button class="modal-close" @click="closeDetail" aria-label="Cerrar">✕</button>
          <div class="modal-header">
            <span :class="['badge', 'badge--lg', selectedConflict.status.toLowerCase()]">
              {{ statusLabel(selectedConflict.status) }}
            </span>
            <h2>{{ selectedConflict.name }}</h2>
            <p class="modal-meta">
              📍 {{ selectedConflict.region || 'Global' }} &nbsp;·&nbsp;
              📅 {{ formatDate(selectedConflict.startDate) }}
            </p>
          </div>
          <p class="modal-description">{{ selectedConflict.description || 'Sin descripción disponible.' }}</p>

          <!-- Mini timeline visual -->
          <div class="modal-timeline">
            <div class="timeline-bar">
              <div
                class="timeline-fill"
                :class="selectedConflict.status.toLowerCase()"
                :style="{ width: timelineWidth(selectedConflict.startDate) + '%' }"
              ></div>
            </div>
            <div class="timeline-labels">
              <span>{{ formatDate(selectedConflict.startDate) }}</span>
              <span class="timeline-duration">{{ durationLabel(selectedConflict.startDate) }}</span>
              <span>Hoy</span>
            </div>
          </div>

          <div v-if="selectedConflict.countries && selectedConflict.countries.length" class="modal-flags">
            <h4>Países involucrados</h4>
            <div class="flags-row">
              <div
                v-for="country in selectedConflict.countries"
                :key="country.id"
                class="flag-chip flag-chip--lg"
              >
                <img :src="getFlagUrl(country.code)" :alt="country.name" class="flag-img" @error="handleFlagError"/>
                <span>{{ country.name || country.code }}</span>
              </div>
            </div>
          </div>

          <!-- ID del servidor (debug útil, oculto en producción) -->
          <p class="modal-id">ID: {{ selectedConflict.id }}</p>
        </div>
      </div>
    </Transition>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, onUnmounted } from 'vue'
import axios from 'axios'

// ─── CONFIGURACIÓN ──────────────────────────────────────────────────────────
const API_URL = "https://conflict-tracker-bueno-production.up.railway.app/api/v1/conflicts"
const today = new Date().toISOString().split('T')[0]

// ─── ESTADO GLOBAL ──────────────────────────────────────────────────────────
const conflicts        = ref([])
const loading          = ref(true)
const loadError        = ref(false)
const creating         = ref(false)
const selectedConflict = ref(null)
const toast            = ref({ visible: false, message: '', type: 'success' })

// ─── FILTROS ────────────────────────────────────────────────────────────────
const filters = ref({
  search:   '',
  statuses: [],
  region:   '',
  sortBy:   'date_desc',
})

const statusOptions = [
  { value: 'ACTIVE',   label: 'Activo',   cls: 'pill--red'    },
  { value: 'TENSION',  label: 'Tensión',  cls: 'pill--yellow' },
  { value: 'RESOLVED', label: 'Resuelto', cls: 'pill--green'  },
]

const hasActiveFilters = computed(() =>
  filters.value.search || filters.value.statuses.length || filters.value.region
)

const uniqueRegions = computed(() => {
  const regions = conflicts.value.map(c => c.region).filter(Boolean)
  return [...new Set(regions)].sort()
})

const filteredConflicts = computed(() => {
  let list = [...conflicts.value]
  const q = filters.value.search.toLowerCase().trim()
  if (q) {
    list = list.filter(c =>
      c.name?.toLowerCase().includes(q) ||
      c.region?.toLowerCase().includes(q) ||
      c.description?.toLowerCase().includes(q)
    )
  }
  if (filters.value.statuses.length) {
    list = list.filter(c => filters.value.statuses.includes(c.status))
  }
  if (filters.value.region) {
    list = list.filter(c => c.region === filters.value.region)
  }
  const sortFns = {
    date_desc: (a, b) => new Date(b.startDate) - new Date(a.startDate),
    date_asc:  (a, b) => new Date(a.startDate) - new Date(b.startDate),
    name_asc:  (a, b) => a.name.localeCompare(b.name),
    name_desc: (a, b) => b.name.localeCompare(a.name),
  }
  list.sort(sortFns[filters.value.sortBy] || sortFns.date_desc)
  return list
})

// ─── STATS REGIONALES para el mini mapa ─────────────────────────────────────
const regionStats = computed(() => {
  const map = {}
  for (const c of conflicts.value) {
    const r = c.region || 'Sin región'
    if (!map[r]) map[r] = { name: r, total: 0, active: 0, tension: 0, resolved: 0 }
    map[r].total++
    if (c.status === 'ACTIVE')   map[r].active++
    if (c.status === 'TENSION')  map[r].tension++
    if (c.status === 'RESOLVED') map[r].resolved++
  }
  return Object.values(map).sort((a, b) => b.total - a.total).slice(0, 8)
})

const maxRegionTotal = computed(() => Math.max(1, ...regionStats.value.map(r => r.total)))

function regionBarWidth(total) {
  return Math.round((total / maxRegionTotal.value) * 100)
}

const regionColorPalette = [
  '#3b82f6', '#f59e0b', '#ef4444', '#8b5cf6',
  '#06b6d4', '#10b981', '#f97316', '#ec4899',
]
const regionColorCache = {}
let regionColorIdx = 0

function regionColor(name) {
  if (!regionColorCache[name]) {
    regionColorCache[name] = regionColorPalette[regionColorIdx % regionColorPalette.length]
    regionColorIdx++
  }
  return regionColorCache[name]
}

// ─── FORMULARIO ─────────────────────────────────────────────────────────────
const newConflict = ref(emptyForm())
const formErrors  = ref({})

function emptyForm() {
  return { name: '', region: '', status: 'ACTIVE', description: '', startDate: today }
}

function validateField(field) {
  const errors = {}
  if (field === 'name' || !field) {
    if (!newConflict.value.name.trim()) errors.name = 'El nombre es obligatorio.'
    else if (newConflict.value.name.trim().length < 3) errors.name = 'Mínimo 3 caracteres.'
  }
  if (field === 'startDate' || !field) {
    if (!newConflict.value.startDate) errors.startDate = 'La fecha es obligatoria.'
    else if (newConflict.value.startDate > today) errors.startDate = 'La fecha no puede ser futura.'
  }
  if (field) {
    formErrors.value = { ...formErrors.value, [field]: errors[field] }
  } else {
    formErrors.value = errors
  }
  return Object.keys(errors).length === 0
}

const isFormValid = computed(() =>
  newConflict.value.name.trim().length >= 3 && !!newConflict.value.startDate
)

// ─── API ─────────────────────────────────────────────────────────────────────

// ✅ FIX #3: Timestamp en cada GET para evitar caché del navegador
const fetchConflicts = async () => {
  loading.value   = true
  loadError.value = false
  try {
    const { data } = await axios.get(API_URL, {
      params: { _t: Date.now() },           // <-- cache-buster
      headers: { 'Cache-Control': 'no-cache' }
    })
    // ✅ FIX #2: Reemplazamos el array entero — Vue detecta el cambio reactivo correctamente
    conflicts.value = Array.isArray(data) ? data : []
  } catch (err) {
    console.error("Error cargando datos:", err)
    loadError.value = true
  } finally {
    loading.value = false
  }
}

// ✅ FIX #1: async/await + fetchConflicts() llamado solo DESPUÉS del 200/201
// ✅ FIX #4: El objeto local NO se pinta antes de recibir respuesta del backend.
//            Solo se actualiza el array tras el GET con IDs generados por la DB.
const createConflict = async () => {
  if (!validateField(null)) return
  if (creating.value) return  // doble-submit guard

  creating.value = true
  try {
    await axios.post(API_URL, {
      name:        newConflict.value.name.trim(),
      region:      newConflict.value.region.trim(),
      status:      newConflict.value.status,
      description: newConflict.value.description.trim(),
      startDate:   newConflict.value.startDate,
    })
    // Solo tras confirmación 2xx limpiamos el formulario y recargamos
    newConflict.value = emptyForm()
    formErrors.value  = {}
    await fetchConflicts()  // <-- trae los datos con IDs reales del servidor
    showToast('Conflicto publicado correctamente.', 'success')
  } catch (err) {
    console.error("Error creando conflicto:", err)
    const msg = err.response?.data?.message || 'Error al guardar. Revisa la conexión.'
    showToast(msg, 'error')
  } finally {
    creating.value = false
  }
}

// ─── HELPERS ─────────────────────────────────────────────────────────────────
function countByStatus(status) {
  return conflicts.value.filter(c => c.status === status).length
}

function statusLabel(status) {
  return { ACTIVE: '● Activo', TENSION: '◆ Tensión', RESOLVED: '✓ Resuelto' }[status] ?? status
}

function getFlagUrl(code) {
  if (!code) return ''
  return `https://flagcdn.com/w40/${code.substring(0, 2).toLowerCase()}.png`
}

function handleFlagError(e) {
  e.target.src = 'https://flagcdn.com/w40/un.png'
}

function formatDate(dateStr) {
  if (!dateStr) return '—'
  return new Date(dateStr).toLocaleDateString('es-ES', { day: '2-digit', month: 'short', year: 'numeric' })
}

function timeAgo(dateStr) {
  if (!dateStr) return ''
  const diff  = Date.now() - new Date(dateStr)
  const days  = Math.floor(diff / 86400000)
  if (days === 0)  return 'Hoy'
  if (days === 1)  return 'Ayer'
  if (days < 30)   return `Hace ${days}d`
  const months = Math.floor(days / 30)
  if (months < 12) return `Hace ${months}m`
  return `Hace ${Math.floor(months / 12)}a`
}

function durationLabel(dateStr) {
  if (!dateStr) return ''
  const days = Math.floor((Date.now() - new Date(dateStr)) / 86400000)
  if (days === 0) return 'Hoy'
  if (days < 30)  return `${days} días`
  const months = Math.floor(days / 30)
  if (months < 12) return `${months} meses`
  const years = Math.floor(months / 12)
  const rem   = months % 12
  return rem > 0 ? `${years}a ${rem}m` : `${years} año${years > 1 ? 's' : ''}`
}

function timelineWidth(dateStr) {
  if (!dateStr) return 10
  const diff   = Date.now() - new Date(dateStr)
  const days   = Math.floor(diff / 86400000)
  // Escala logarítmica: 1 día = 5%, 365 días = 100%
  const pct = Math.min(100, Math.max(5, (Math.log(days + 1) / Math.log(366)) * 100))
  return Math.round(pct)
}

function toggleStatusFilter(value) {
  const idx = filters.value.statuses.indexOf(value)
  if (idx === -1) filters.value.statuses.push(value)
  else filters.value.statuses.splice(idx, 1)
}

function resetFilters() {
  filters.value = { search: '', statuses: [], region: '', sortBy: 'date_desc' }
}

function openDetail(conflict) {
  selectedConflict.value = conflict
  document.body.style.overflow = 'hidden'
}

function closeDetail() {
  selectedConflict.value = null
  document.body.style.overflow = ''
}

let toastTimer = null
function showToast(message, type = 'success') {
  clearTimeout(toastTimer)
  toast.value = { visible: true, message, type }
  toastTimer = setTimeout(() => { toast.value.visible = false }, 3500)
}

// ─── LIFECYCLE ────────────────────────────────────────────────────────────────
onMounted(fetchConflicts)

const escHandler = (e) => { if (e.key === 'Escape') closeDetail() }
if (typeof window !== 'undefined') window.addEventListener('keydown', escHandler)
onUnmounted(() => {
  if (typeof window !== 'undefined') window.removeEventListener('keydown', escHandler)
  clearTimeout(toastTimer)
})
</script>

<style scoped>
/* ═══════════════════════════════════════════
   VARIABLES
═══════════════════════════════════════════ */
.app-container {
  --bg:          #080d14;
  --surface:     #0e1621;
  --surface-2:   #152030;
  --border:      rgba(255,255,255,0.07);
  --border-2:    rgba(255,255,255,0.12);
  --text:        #e2eaf4;
  --text-muted:  #556578;
  --text-dim:    #8899aa;
  --accent:      #3b82f6;
  --accent-glow: rgba(59,130,246,0.15);
  --red:         #f87171;
  --red-bg:      rgba(248,113,113,0.1);
  --yellow:      #fbbf24;
  --yellow-bg:   rgba(251,191,36,0.1);
  --green:       #34d399;
  --green-bg:    rgba(52,211,153,0.1);
  --radius:      14px;
  --radius-sm:   8px;
  --shadow:      0 8px 32px rgba(0,0,0,0.4);

  font-family: 'DM Sans', 'Segoe UI', sans-serif;
  background: var(--bg);
  color: var(--text);
  min-height: 100vh;
  padding: 2rem 2.5rem;
}

/* ═══════════════════════════════════════════
   HEADER
═══════════════════════════════════════════ */
.main-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  gap: 1.5rem;
  margin-bottom: 2.5rem;
  padding-bottom: 1.5rem;
  border-bottom: 1px solid var(--border);
}

.header-left { display: flex; align-items: center; gap: 1rem; }

.logo-mark {
  width: 44px;
  height: 44px;
  background: var(--accent-glow);
  border: 1px solid rgba(59,130,246,0.3);
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}

.globe-svg { width: 26px; height: 26px; color: var(--accent); }
.main-header h1 { font-size: 1.4rem; font-weight: 700; margin: 0; letter-spacing: -0.02em; }
.subtitle { font-size: 0.78rem; color: var(--text-muted); margin: 2px 0 0; }

.header-right { display: flex; align-items: center; gap: 1rem; flex-wrap: wrap; }

.live-chip {
  display: flex;
  align-items: center;
  gap: 6px;
  font-size: 0.65rem;
  font-weight: 800;
  letter-spacing: 0.1em;
  color: var(--green);
  border: 1px solid rgba(52,211,153,0.3);
  background: rgba(52,211,153,0.07);
  padding: 5px 10px;
  border-radius: 20px;
}

.pulse-dot {
  width: 7px;
  height: 7px;
  background: var(--green);
  border-radius: 50%;
  animation: pulse 1.8s ease-in-out infinite;
}

.stats-chips { display: flex; gap: 0.5rem; }

.chip {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 6px 14px;
  border-radius: var(--radius-sm);
  border: 1px solid var(--border);
}

.chip-value { font-size: 1.1rem; font-weight: 700; line-height: 1; }
.chip-label { font-size: 0.6rem; color: var(--text-muted); margin-top: 2px; }
.chip--red    { border-color: rgba(248,113,113,0.3); background: var(--red-bg); }
.chip--red .chip-value    { color: var(--red); }
.chip--yellow { border-color: rgba(251,191,36,0.3);  background: var(--yellow-bg); }
.chip--yellow .chip-value { color: var(--yellow); }
.chip--green  { border-color: rgba(52,211,153,0.3);  background: var(--green-bg); }
.chip--green .chip-value  { color: var(--green); }

/* ═══════════════════════════════════════════
   LAYOUT
═══════════════════════════════════════════ */
.content {
  display: grid;
  grid-template-columns: 320px 1fr;
  gap: 2rem;
  align-items: start;
}

@media (max-width: 960px) {
  .content { grid-template-columns: 1fr; }
  .app-container { padding: 1.2rem; }
}

.sidebar { display: flex; flex-direction: column; gap: 1.5rem; position: sticky; top: 2rem; }

/* ═══════════════════════════════════════════
   CARDS
═══════════════════════════════════════════ */
.card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 1.5rem;
}

.card-title {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 0.85rem;
  font-weight: 600;
  color: var(--text-dim);
  text-transform: uppercase;
  letter-spacing: 0.06em;
  margin: 0 0 1.2rem;
}

.icon { width: 16px; height: 16px; flex-shrink: 0; }

/* ═══════════════════════════════════════════
   MAPA REGIONAL (nuevo componente)
═══════════════════════════════════════════ */
.map-card { padding: 1.25rem 1.5rem; }

.region-bars { display: flex; flex-direction: column; gap: 10px; }

.region-row {
  display: grid;
  grid-template-columns: 90px 1fr 56px;
  align-items: center;
  gap: 8px;
}

.region-name {
  font-size: 0.72rem;
  color: var(--text-dim);
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.region-track {
  height: 6px;
  background: var(--surface-2);
  border-radius: 3px;
  overflow: hidden;
}

.region-fill {
  height: 100%;
  border-radius: 3px;
  transition: width 0.6s cubic-bezier(0.16, 1, 0.3, 1);
  opacity: 0.85;
}

.region-dots {
  display: flex;
  gap: 4px;
  justify-content: flex-end;
}

.rdot {
  font-size: 0.6rem;
  font-weight: 700;
  width: 16px;
  height: 16px;
  border-radius: 4px;
  display: flex;
  align-items: center;
  justify-content: center;
  line-height: 1;
}

.rdot--red    { background: var(--red-bg);    color: var(--red); }
.rdot--yellow { background: var(--yellow-bg); color: var(--yellow); }
.rdot--green  { background: var(--green-bg);  color: var(--green); }

.region-empty { font-size: 0.75rem; color: var(--text-muted); text-align: center; padding: 1rem 0; }

/* ═══════════════════════════════════════════
   FILTROS
═══════════════════════════════════════════ */
.filter-group { margin-bottom: 1rem; }

.filter-group label {
  display: block;
  font-size: 0.72rem;
  font-weight: 600;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin-bottom: 0.5rem;
}

.search-wrap { position: relative; display: flex; align-items: center; }

.search-icon {
  position: absolute;
  left: 10px;
  width: 14px;
  height: 14px;
  color: var(--text-muted);
  pointer-events: none;
}

.search-input {
  width: 100%;
  padding: 0.65rem 2rem 0.65rem 2.2rem;
  background: var(--surface-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  color: var(--text);
  font-size: 0.85rem;
  transition: border-color 0.2s;
}

.clear-btn {
  position: absolute;
  right: 8px;
  background: none;
  border: none;
  color: var(--text-muted);
  cursor: pointer;
  font-size: 0.8rem;
  padding: 2px;
  line-height: 1;
}

.filter-pills { display: flex; gap: 6px; flex-wrap: wrap; }

.pill {
  padding: 4px 12px;
  border-radius: 20px;
  font-size: 0.72rem;
  font-weight: 600;
  border: 1px solid var(--border);
  background: transparent;
  color: var(--text-dim);
  cursor: pointer;
  transition: all 0.2s;
}

.pill--red.active    { background: var(--red-bg);    border-color: var(--red);    color: var(--red); }
.pill--yellow.active { background: var(--yellow-bg); border-color: var(--yellow); color: var(--yellow); }
.pill--green.active  { background: var(--green-bg);  border-color: var(--green);  color: var(--green); }
.pill:hover          { border-color: var(--border-2); color: var(--text); }

.btn-reset {
  width: 100%;
  padding: 0.55rem;
  background: transparent;
  border: 1px dashed var(--border-2);
  border-radius: var(--radius-sm);
  color: var(--text-muted);
  font-size: 0.8rem;
  cursor: pointer;
  transition: all 0.2s;
  margin-top: 0.25rem;
}
.btn-reset:hover { border-color: var(--red); color: var(--red); }

/* ═══════════════════════════════════════════
   FORMULARIO
═══════════════════════════════════════════ */
.conflict-form { display: flex; flex-direction: column; gap: 1rem; }
.input-group { display: flex; flex-direction: column; gap: 0.4rem; }
.input-group.error input,
.input-group.error textarea { border-color: var(--red); }
.input-row { display: grid; grid-template-columns: 1fr 1fr; gap: 0.75rem; }

label {
  font-size: 0.72rem;
  font-weight: 600;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
}

.required { color: var(--red); }
.error-msg { font-size: 0.7rem; color: var(--red); }
.char-count { font-size: 0.65rem; color: var(--text-muted); text-align: right; }

input, select, textarea, .select-input {
  background: var(--surface-2);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  padding: 0.65rem 0.8rem;
  color: var(--text);
  font-size: 0.85rem;
  transition: border-color 0.2s, box-shadow 0.2s;
  width: 100%;
  box-sizing: border-box;
}

input:focus, select:focus, textarea:focus {
  border-color: var(--accent);
  box-shadow: 0 0 0 3px var(--accent-glow);
  outline: none;
}

input:disabled, select:disabled, textarea:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

textarea { resize: vertical; font-family: inherit; }

.btn-submit {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  padding: 0.8rem;
  background: var(--accent);
  color: #fff;
  font-size: 0.88rem;
  font-weight: 600;
  border: none;
  border-radius: var(--radius-sm);
  cursor: pointer;
  transition: all 0.2s;
  margin-top: 0.25rem;
}

.btn-submit:hover:not(:disabled) {
  background: #60a5fa;
  transform: translateY(-1px);
  box-shadow: 0 4px 20px rgba(59,130,246,0.3);
}

.btn-submit:disabled { opacity: 0.5; cursor: not-allowed; }

.btn-spinner {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255,255,255,0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

/* ═══════════════════════════════════════════
   FEED HEADER
═══════════════════════════════════════════ */
.feed-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 1.2rem;
}

.result-count { font-size: 0.82rem; color: var(--text-muted); margin: 0; }
.result-count strong { color: var(--text); }

.btn-refresh {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 6px 14px;
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius-sm);
  color: var(--text-dim);
  font-size: 0.78rem;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-refresh:hover:not(:disabled) { border-color: var(--border-2); color: var(--text); }
.btn-refresh:disabled { opacity: 0.5; cursor: not-allowed; }
.btn-refresh.spinning .icon { animation: spin 1s linear infinite; }

/* ═══════════════════════════════════════════
   GRID DE CONFLICTOS
═══════════════════════════════════════════ */
.grid-layout {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.2rem;
}

.conflict-card {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  cursor: pointer;
  transition: transform 0.2s, border-color 0.2s, box-shadow 0.2s;
  position: relative;
  overflow: hidden;
}

.conflict-card::before {
  content: '';
  position: absolute;
  top: 0; left: 0; right: 0;
  height: 3px;
  opacity: 0;
  transition: opacity 0.2s;
}

.conflict-card.status-active::before   { background: var(--red); }
.conflict-card.status-tension::before  { background: var(--yellow); }
.conflict-card.status-resolved::before { background: var(--green); }

.conflict-card:hover {
  transform: translateY(-3px);
  border-color: var(--border-2);
  box-shadow: var(--shadow);
}
.conflict-card:hover::before { opacity: 1; }

.card-top { flex: 1; }

.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 0.75rem;
}

.card-age { font-size: 0.65rem; color: var(--text-muted); }

.card-title-conflict {
  font-size: 1rem;
  font-weight: 600;
  color: var(--text);
  margin: 0 0 0.6rem;
  letter-spacing: -0.01em;
  line-height: 1.3;
}

.description {
  font-size: 0.82rem;
  color: var(--text-dim);
  line-height: 1.6;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-footer {
  margin-top: 1.2rem;
  padding-top: 1rem;
  border-top: 1px solid var(--border);
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.meta { display: flex; gap: 1rem; flex-wrap: wrap; }

.meta-item {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 0.72rem;
  color: var(--text-muted);
}

.meta-icon { width: 12px; height: 12px; }

/* ═══════════════════════════════════════════
   BADGES
═══════════════════════════════════════════ */
.badge {
  font-size: 0.62rem;
  font-weight: 700;
  padding: 3px 8px;
  border-radius: 5px;
  letter-spacing: 0.04em;
  white-space: nowrap;
}

.badge--lg { font-size: 0.72rem; padding: 5px 12px; }
.active   { background: var(--red-bg);    color: var(--red);    border: 1px solid rgba(248,113,113,0.3); }
.tension  { background: var(--yellow-bg); color: var(--yellow); border: 1px solid rgba(251,191,36,0.3); }
.resolved { background: var(--green-bg);  color: var(--green);  border: 1px solid rgba(52,211,153,0.3); }

/* ═══════════════════════════════════════════
   FLAGS
═══════════════════════════════════════════ */
.flags-row { display: flex; flex-wrap: wrap; gap: 6px; align-items: center; }

.flag-chip {
  display: flex;
  align-items: center;
  gap: 5px;
  background: var(--surface-2);
  border: 1px solid var(--border);
  padding: 3px 7px;
  border-radius: 5px;
  font-size: 0.7rem;
  color: var(--text-dim);
}

.flag-chip--lg { padding: 5px 10px; font-size: 0.8rem; }
.flag-img { width: 18px; height: auto; border-radius: 2px; }
.flag-chip--lg .flag-img { width: 22px; }

.flag-overflow {
  font-size: 0.7rem;
  color: var(--text-muted);
  padding: 3px 7px;
  background: var(--surface-2);
  border-radius: 5px;
  border: 1px dashed var(--border);
}

/* ═══════════════════════════════════════════
   ESTADOS VACÍOS / ERROR
═══════════════════════════════════════════ */
.state-msg {
  grid-column: 1 / -1;
  text-align: center;
  padding: 5rem 2rem;
  color: var(--text-muted);
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 0.75rem;
}

.state-icon { width: 40px; height: 40px; opacity: 0.4; }

.btn-retry {
  padding: 7px 18px;
  background: var(--surface-2);
  border: 1px solid var(--border-2);
  border-radius: var(--radius-sm);
  color: var(--text-dim);
  font-size: 0.82rem;
  cursor: pointer;
  transition: all 0.2s;
}
.btn-retry:hover { color: var(--text); border-color: var(--accent); }

/* SKELETON */
.skeleton-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
  gap: 1.2rem;
  width: 100%;
}

.skeleton-card {
  background: var(--surface);
  border: 1px solid var(--border);
  border-radius: var(--radius);
  padding: 1.5rem;
  display: flex;
  flex-direction: column;
  gap: 0.8rem;
}

.skeleton-line {
  height: 12px;
  border-radius: 6px;
  background: linear-gradient(90deg, var(--surface-2) 25%, var(--border) 50%, var(--surface-2) 75%);
  background-size: 200% 100%;
  animation: shimmer 1.5s infinite;
}

.skeleton-line.short  { width: 40%; }
.skeleton-line.medium { width: 70%; }

/* ═══════════════════════════════════════════
   MODAL
═══════════════════════════════════════════ */
.modal-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0,0,0,0.7);
  backdrop-filter: blur(4px);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 1000;
  padding: 1.5rem;
}

.modal {
  background: var(--surface);
  border: 1px solid var(--border-2);
  border-radius: 20px;
  padding: 2rem;
  max-width: 560px;
  width: 100%;
  box-shadow: 0 24px 64px rgba(0,0,0,0.6);
  position: relative;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-close {
  position: absolute;
  top: 1rem; right: 1rem;
  background: var(--surface-2);
  border: 1px solid var(--border);
  width: 30px; height: 30px;
  border-radius: 50%;
  color: var(--text-muted);
  cursor: pointer;
  font-size: 0.8rem;
  display: flex;
  align-items: center;
  justify-content: center;
  transition: all 0.2s;
}
.modal-close:hover { color: var(--text); border-color: var(--border-2); }

.modal-header { margin-bottom: 1.5rem; }
.modal-header h2 { font-size: 1.4rem; font-weight: 700; margin: 0.75rem 0 0.5rem; }
.modal-meta { font-size: 0.8rem; color: var(--text-muted); margin: 0; }
.modal-description { font-size: 0.9rem; color: var(--text-dim); line-height: 1.7; }
.modal-id { font-size: 0.65rem; color: var(--text-muted); margin-top: 1.5rem; opacity: 0.5; }

/* Timeline del modal */
.modal-timeline {
  margin: 1.25rem 0;
  padding: 1rem;
  background: var(--surface-2);
  border-radius: var(--radius-sm);
  border: 1px solid var(--border);
}

.timeline-bar {
  height: 8px;
  background: var(--border);
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 0.5rem;
}

.timeline-fill {
  height: 100%;
  border-radius: 4px;
  transition: width 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}

.timeline-fill.active   { background: var(--red); }
.timeline-fill.tension  { background: var(--yellow); }
.timeline-fill.resolved { background: var(--green); }

.timeline-labels {
  display: flex;
  justify-content: space-between;
  font-size: 0.65rem;
  color: var(--text-muted);
}

.timeline-duration {
  font-weight: 600;
  color: var(--text-dim);
}

.modal-flags { margin-top: 1.5rem; }
.modal-flags h4 {
  font-size: 0.72rem;
  font-weight: 600;
  color: var(--text-muted);
  text-transform: uppercase;
  letter-spacing: 0.05em;
  margin: 0 0 0.75rem;
}

/* ═══════════════════════════════════════════
   TOAST
═══════════════════════════════════════════ */
.toast {
  position: fixed;
  bottom: 2rem; right: 2rem;
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 0.8rem 1.2rem;
  border-radius: var(--radius-sm);
  font-size: 0.85rem;
  font-weight: 500;
  z-index: 9999;
  box-shadow: var(--shadow);
  border: 1px solid;
}

.toast.success { background: var(--green-bg); border-color: var(--green); color: var(--green); }
.toast.error   { background: var(--red-bg);   border-color: var(--red);   color: var(--red); }
.toast-icon    { font-weight: 700; font-size: 1rem; }

/* ═══════════════════════════════════════════
   ANIMACIONES
═══════════════════════════════════════════ */
@keyframes pulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50%       { opacity: 0.4; transform: scale(0.9); }
}
@keyframes spin { to { transform: rotate(360deg); } }
@keyframes shimmer {
  0%   { background-position: 200% 0; }
  100% { background-position: -200% 0; }
}

.card-list-enter-active { transition: all 0.35s ease; }
.card-list-leave-active { transition: all 0.25s ease; }
.card-list-enter-from   { opacity: 0; transform: translateY(16px); }
.card-list-leave-to     { opacity: 0; transform: scale(0.95); }
.card-list-move         { transition: transform 0.35s ease; }

.modal-enter-active, .modal-leave-active { transition: all 0.25s ease; }
.modal-enter-from, .modal-leave-to       { opacity: 0; }
.modal-enter-active .modal, .modal-leave-active .modal { transition: all 0.25s ease; }
.modal-enter-from .modal  { transform: scale(0.96) translateY(8px); }
.modal-leave-to .modal    { transform: scale(0.96) translateY(8px); }

.toast-enter-active, .toast-leave-active { transition: all 0.3s ease; }
.toast-enter-from, .toast-leave-to       { opacity: 0; transform: translateX(12px); }

::-webkit-scrollbar       { width: 5px; }
::-webkit-scrollbar-track { background: var(--bg); }
::-webkit-scrollbar-thumb { background: var(--surface-2); border-radius: 3px; }
</style>