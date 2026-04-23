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
  <div class="app-root">

    <!-- Fondo animado con cuadrícula táctica -->
    <div class="bg-grid"></div>
    <div class="bg-glow"></div>

    <!-- Header -->
    <header class="main-header">
      <div class="header-inner">
        <div class="header-brand">
          <div class="brand-icon">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5">
              <circle cx="12" cy="12" r="9"/>
              <path d="M12 3v2M12 19v2M3 12h2M19 12h2"/>
              <circle cx="12" cy="12" r="3" fill="currentColor"/>
            </svg>
          </div>
          <div>
            <h1 class="brand-title">CONFLICT MONITOR</h1>
            <p class="brand-sub">Sistema de seguimiento global</p>
          </div>
        </div>
        <div class="header-stats">
          <div class="stat-chip">
            <span class="stat-dot dot-active"></span>
            <span>{{ conflicts.filter(c => c.status === 'ACTIVE').length }} Activos</span>
          </div>
          <div class="stat-chip">
            <span class="stat-dot dot-tension"></span>
            <span>{{ conflicts.filter(c => c.status === 'TENSION').length }} En tensión</span>
          </div>
          <div class="stat-chip">
            <span class="stat-dot dot-resolved"></span>
            <span>{{ conflicts.filter(c => c.status === 'RESOLVED').length }} Resueltos</span>
          </div>
        </div>
      </div>
    </header>

    <!-- Layout -->
    <div class="layout">

      <!-- Sidebar -->
      <aside class="sidebar">
        <div class="panel">
          <div class="panel-header">
            <span class="panel-label">NUEVO REGISTRO</span>
          </div>
          <form @submit.prevent="createConflict" class="conflict-form">

            <div class="field-group">
              <label class="field-label">Título del conflicto</label>
              <input class="field-input" v-model="newConflict.name" placeholder="Ej. Crisis en el Mediterráneo..." required />
            </div>

            <div class="field-row">
              <div class="field-group">
                <label class="field-label">Región</label>
                <input class="field-input" v-model="newConflict.region" placeholder="Región..." />
              </div>
              <div class="field-group">
                <label class="field-label">Estado</label>
                <select class="field-input field-select" v-model="newConflict.status">
                  <option value="ACTIVE">ACTIVO</option>
                  <option value="TENSION">TENSIÓN</option>
                  <option value="RESOLVED">RESUELTO</option>
                </select>
              </div>
            </div>

            <div class="field-group">
              <label class="field-label">Descripción</label>
              <textarea class="field-input field-textarea" v-model="newConflict.description" placeholder="Describe los detalles del conflicto..."></textarea>
            </div>

            <button type="submit" :disabled="isSubmitting" class="submit-btn">
              <span v-if="isSubmitting" class="btn-spinner"></span>
              <span v-else>
                <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M12 5v14M5 12l7 7 7-7"/></svg>
              </span>
              {{ isSubmitting ? 'Enviando...' : 'Publicar registro' }}
            </button>
          </form>
        </div>

        <!-- Leyenda de estados -->
        <div class="legend-panel">
          <p class="legend-title">ESTADOS</p>
          <div class="legend-item"><span class="legend-dot dot-active"></span><span>Activo — conflicto en curso</span></div>
          <div class="legend-item"><span class="legend-dot dot-tension"></span><span>Tensión — situación frágil</span></div>
          <div class="legend-item"><span class="legend-dot dot-resolved"></span><span>Resuelto — conflicto cerrado</span></div>
        </div>
      </aside>

      <!-- Feed principal -->
      <section class="feed">

        <!-- Loading state -->
        <div v-if="loading" class="loading-state">
          <div class="radar-ring"></div>
          <div class="radar-ring r2"></div>
          <div class="radar-ring r3"></div>
          <p class="loading-text">Obteniendo registros...</p>
        </div>

        <!-- Lista de conflictos -->
        <template v-else>
          <div class="feed-header">
            <span class="feed-count">{{ conflicts.length }} registros</span>
            <span class="feed-line"></span>
          </div>

          <div class="conflict-grid">
            <div
              v-for="c in conflicts"
              :key="c.id"
              :class="['conflict-card', (c.status || 'ACTIVE').toLowerCase()]"
            >
              <!-- Acento lateral -->
              <div class="card-accent"></div>

              <div class="card-body">
                <div class="card-top">
                  <span :class="['status-badge', (c.status || 'ACTIVE').toLowerCase()]">
                    <span class="badge-dot"></span>
                    {{ c.status || 'ACTIVE' }}
                  </span>
                  <span class="card-region" v-if="c.region">
                    <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0118 0z"/><circle cx="12" cy="10" r="3"/></svg>
                    {{ c.region }}
                  </span>
                </div>

                <h3 class="card-title">{{ c.name }}</h3>
                <p class="card-desc" v-if="c.description">{{ c.description }}</p>

                <div class="card-footer">
                  <svg width="11" height="11" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><rect x="3" y="4" width="18" height="18" rx="2"/><path d="M16 2v4M8 2v4M3 10h18"/></svg>
                  <span>{{ c.startDate }}</span>
                </div>
              </div>
            </div>
          </div>
        </template>
      </section>
    </div>

    <!-- Toast -->
    <Transition name="toast-slide">
      <div v-if="toast.visible" :class="['toast-msg', toast.type]">
        <span v-if="toast.type === 'success'">
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M20 6L9 17l-5-5"/></svg>
        </span>
        <span v-else>
          <svg width="14" height="14" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.5"><circle cx="12" cy="12" r="10"/><path d="M15 9l-6 6M9 9l6 6"/></svg>
        </span>
        {{ toast.message }}
      </div>
    </Transition>
  </div>
</template>

<style scoped>
/* ─────────────── RESET & BASE ─────────────── */
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

/* ─────────────── TOKENS ─────────────── */
:root {
  --c-bg: #080c14;
  --c-surface: #0d1421;
  --c-surface2: #111a2e;
  --c-border: rgba(255,255,255,0.07);
  --c-border-strong: rgba(255,255,255,0.14);
  --c-text: #e8edf7;
  --c-text-muted: #5c7094;
  --c-text-dim: #3a4d6b;
  --c-accent: #2563eb;
  --c-active: #ef4444;
  --c-tension: #f59e0b;
  --c-resolved: #10b981;
  --c-active-bg: rgba(239,68,68,0.08);
  --c-tension-bg: rgba(245,158,11,0.08);
  --c-resolved-bg: rgba(16,185,129,0.08);
  --font: 'JetBrains Mono', 'Fira Code', 'Courier New', monospace;
  --font-sans: 'Inter', system-ui, sans-serif;
}

@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600&family=Inter:wght@400;500;600&display=swap');

/* ─────────────── FONDO ─────────────── */
.app-root {
  background: var(--c-bg);
  color: var(--c-text);
  min-height: 100vh;
  font-family: var(--font-sans);
  position: relative;
  overflow-x: hidden;
}

.bg-grid {
  position: fixed;
  inset: 0;
  background-image:
    linear-gradient(rgba(37,99,235,0.04) 1px, transparent 1px),
    linear-gradient(90deg, rgba(37,99,235,0.04) 1px, transparent 1px);
  background-size: 48px 48px;
  pointer-events: none;
  z-index: 0;
}

.bg-glow {
  position: fixed;
  top: -20%;
  left: 50%;
  transform: translateX(-50%);
  width: 800px;
  height: 500px;
  background: radial-gradient(ellipse, rgba(37,99,235,0.12) 0%, transparent 70%);
  pointer-events: none;
  z-index: 0;
}

/* ─────────────── HEADER ─────────────── */
.main-header {
  position: relative;
  z-index: 10;
  border-bottom: 1px solid var(--c-border);
  background: rgba(8,12,20,0.9);
  backdrop-filter: blur(12px);
  padding: 0 32px;
}

.header-inner {
  max-width: 1400px;
  margin: 0 auto;
  height: 72px;
  display: flex;
  align-items: center;
  justify-content: space-between;
}

.header-brand {
  display: flex;
  align-items: center;
  gap: 16px;
}

.brand-icon {
  width: 40px;
  height: 40px;
  background: rgba(37,99,235,0.15);
  border: 1px solid rgba(37,99,235,0.4);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #60a5fa;
  animation: pulse-icon 3s ease-in-out infinite;
}

@keyframes pulse-icon {
  0%, 100% { box-shadow: 0 0 0 0 rgba(37,99,235,0.3); }
  50% { box-shadow: 0 0 0 6px rgba(37,99,235,0); }
}

.brand-title {
  font-family: var(--font);
  font-size: 14px;
  font-weight: 600;
  letter-spacing: 0.2em;
  color: var(--c-text);
}

.brand-sub {
  font-size: 11px;
  color: var(--c-text-muted);
  letter-spacing: 0.05em;
  margin-top: 1px;
}

.header-stats {
  display: flex;
  gap: 8px;
}

.stat-chip {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 5px 12px;
  background: var(--c-surface);
  border: 1px solid var(--c-border);
  border-radius: 6px;
  font-family: var(--font);
  font-size: 11px;
  font-weight: 500;
  color: var(--c-text-muted);
  letter-spacing: 0.04em;
}

/* ─────────────── LAYOUT ─────────────── */
.layout {
  position: relative;
  z-index: 1;
  max-width: 1400px;
  margin: 0 auto;
  padding: 32px;
  display: grid;
  grid-template-columns: 340px 1fr;
  gap: 24px;
  min-height: calc(100vh - 72px);
}

/* ─────────────── SIDEBAR ─────────────── */
.sidebar {
  display: flex;
  flex-direction: column;
  gap: 16px;
  align-self: start;
  position: sticky;
  top: 32px;
}

.panel {
  background: var(--c-surface);
  border: 1px solid var(--c-border);
  border-radius: 14px;
  overflow: hidden;
}

.panel-header {
  padding: 14px 20px;
  border-bottom: 1px solid var(--c-border);
  background: rgba(37,99,235,0.05);
}

.panel-label {
  font-family: var(--font);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.18em;
  color: #60a5fa;
}

.conflict-form {
  padding: 20px;
  display: flex;
  flex-direction: column;
  gap: 16px;
}

.field-group {
  display: flex;
  flex-direction: column;
  gap: 6px;
}

.field-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

.field-label {
  font-family: var(--font);
  font-size: 10px;
  font-weight: 500;
  letter-spacing: 0.1em;
  color: var(--c-text-muted);
  text-transform: uppercase;
}

.field-input {
  background: var(--c-bg);
  border: 1px solid var(--c-border-strong);
  border-radius: 8px;
  color: var(--c-text);
  padding: 10px 12px;
  font-size: 13px;
  font-family: var(--font-sans);
  transition: border-color 0.2s, box-shadow 0.2s;
  outline: none;
  width: 100%;
}

.field-input:focus {
  border-color: rgba(37,99,235,0.6);
  box-shadow: 0 0 0 3px rgba(37,99,235,0.12);
}

.field-input::placeholder { color: var(--c-text-dim); }
.field-select { cursor: pointer; }
.field-select option { background: #111a2e; }
.field-textarea { resize: vertical; min-height: 90px; line-height: 1.5; }

.submit-btn {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 8px;
  background: var(--c-accent);
  border: none;
  border-radius: 8px;
  color: #fff;
  font-size: 13px;
  font-weight: 600;
  font-family: var(--font);
  letter-spacing: 0.06em;
  padding: 12px;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s, opacity 0.2s;
}

.submit-btn:hover { background: #1d4ed8; }
.submit-btn:active { transform: scale(0.98); }
.submit-btn:disabled { opacity: 0.5; cursor: not-allowed; }

.btn-spinner {
  width: 14px;
  height: 14px;
  border: 2px solid rgba(255,255,255,0.3);
  border-top-color: #fff;
  border-radius: 50%;
  animation: spin 0.7s linear infinite;
}

@keyframes spin { to { transform: rotate(360deg); } }

/* Leyenda */
.legend-panel {
  background: var(--c-surface);
  border: 1px solid var(--c-border);
  border-radius: 14px;
  padding: 16px 20px;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.legend-title {
  font-family: var(--font);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.18em;
  color: var(--c-text-dim);
  margin-bottom: 2px;
}

.legend-item {
  display: flex;
  align-items: center;
  gap: 10px;
  font-size: 12px;
  color: var(--c-text-muted);
}

/* ─────────────── DOTS ─────────────── */
.stat-dot, .legend-dot, .badge-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  display: inline-block;
  flex-shrink: 0;
}

.dot-active { background: var(--c-active); box-shadow: 0 0 6px var(--c-active); }
.dot-tension { background: var(--c-tension); box-shadow: 0 0 6px var(--c-tension); }
.dot-resolved { background: var(--c-resolved); box-shadow: 0 0 6px var(--c-resolved); }

/* ─────────────── FEED ─────────────── */
.feed {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.feed-header {
  display: flex;
  align-items: center;
  gap: 16px;
}

.feed-count {
  font-family: var(--font);
  font-size: 11px;
  letter-spacing: 0.12em;
  color: var(--c-text-muted);
  white-space: nowrap;
}

.feed-line {
  flex: 1;
  height: 1px;
  background: var(--c-border);
}

/* ─────────────── LOADING ─────────────── */
.loading-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 80px 0;
  gap: 12px;
  position: relative;
}

.radar-ring {
  position: absolute;
  width: 60px;
  height: 60px;
  border-radius: 50%;
  border: 1px solid rgba(37,99,235,0.5);
  animation: radar 2s ease-out infinite;
}

.radar-ring.r2 { animation-delay: 0.66s; }
.radar-ring.r3 { animation-delay: 1.33s; }

@keyframes radar {
  0% { transform: scale(0.4); opacity: 1; }
  100% { transform: scale(3); opacity: 0; }
}

.loading-text {
  margin-top: 80px;
  font-family: var(--font);
  font-size: 11px;
  letter-spacing: 0.15em;
  color: var(--c-text-muted);
}

/* ─────────────── CONFLICT GRID ─────────────── */
.conflict-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 16px;
}

.conflict-card {
  background: var(--c-surface);
  border: 1px solid var(--c-border);
  border-radius: 12px;
  overflow: hidden;
  position: relative;
  display: flex;
  transition: transform 0.2s, border-color 0.2s, box-shadow 0.2s;
  animation: card-in 0.4s ease both;
}

@keyframes card-in {
  from { opacity: 0; transform: translateY(12px); }
  to { opacity: 1; transform: translateY(0); }
}

.conflict-card:hover {
  transform: translateY(-2px);
  border-color: var(--c-border-strong);
}

.conflict-card.active { background: var(--c-active-bg); border-color: rgba(239,68,68,0.2); }
.conflict-card.active:hover { border-color: rgba(239,68,68,0.4); box-shadow: 0 8px 32px rgba(239,68,68,0.1); }
.conflict-card.tension { background: var(--c-tension-bg); border-color: rgba(245,158,11,0.2); }
.conflict-card.tension:hover { border-color: rgba(245,158,11,0.4); box-shadow: 0 8px 32px rgba(245,158,11,0.1); }
.conflict-card.resolved { background: var(--c-resolved-bg); border-color: rgba(16,185,129,0.2); }
.conflict-card.resolved:hover { border-color: rgba(16,185,129,0.4); box-shadow: 0 8px 32px rgba(16,185,129,0.1); }

.card-accent {
  width: 3px;
  flex-shrink: 0;
  background: var(--c-border-strong);
  border-radius: 12px 0 0 12px;
}

.conflict-card.active .card-accent { background: var(--c-active); }
.conflict-card.tension .card-accent { background: var(--c-tension); }
.conflict-card.resolved .card-accent { background: var(--c-resolved); }

.card-body {
  padding: 18px;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.card-top {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.status-badge {
  display: flex;
  align-items: center;
  gap: 6px;
  padding: 3px 9px 3px 7px;
  border-radius: 99px;
  font-family: var(--font);
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 0.12em;
}

.status-badge.active { background: rgba(239,68,68,0.12); color: #fca5a5; }
.status-badge.active .badge-dot { background: var(--c-active); animation: blink 1.5s ease-in-out infinite; }
.status-badge.tension { background: rgba(245,158,11,0.12); color: #fcd34d; }
.status-badge.tension .badge-dot { background: var(--c-tension); }
.status-badge.resolved { background: rgba(16,185,129,0.12); color: #6ee7b7; }
.status-badge.resolved .badge-dot { background: var(--c-resolved); }

@keyframes blink {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.3; }
}

.card-region {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 11px;
  color: var(--c-text-muted);
  font-family: var(--font);
}

.card-title {
  font-size: 15px;
  font-weight: 600;
  line-height: 1.35;
  color: var(--c-text);
  letter-spacing: -0.01em;
}

.card-desc {
  font-size: 13px;
  line-height: 1.6;
  color: #8ba0bf;
  display: -webkit-box;
  -webkit-line-clamp: 3;
  -webkit-box-orient: vertical;
  overflow: hidden;
}

.card-footer {
  display: flex;
  align-items: center;
  gap: 6px;
  margin-top: auto;
  padding-top: 10px;
  border-top: 1px solid var(--c-border);
  font-family: var(--font);
  font-size: 11px;
  color: var(--c-text-dim);
}

/* ─────────────── TOAST ─────────────── */
.toast-msg {
  position: fixed;
  bottom: 28px;
  right: 28px;
  z-index: 9999;
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 12px 18px;
  border-radius: 10px;
  font-family: var(--font);
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.06em;
  box-shadow: 0 8px 32px rgba(0,0,0,0.5);
}

.toast-msg.success {
  background: rgba(16,185,129,0.15);
  border: 1px solid rgba(16,185,129,0.35);
  color: #6ee7b7;
}

.toast-msg.error {
  background: rgba(239,68,68,0.15);
  border: 1px solid rgba(239,68,68,0.35);
  color: #fca5a5;
}

.toast-slide-enter-active, .toast-slide-leave-active { transition: all 0.3s cubic-bezier(0.16,1,0.3,1); }
.toast-slide-enter-from { transform: translateY(12px) scale(0.96); opacity: 0; }
.toast-slide-leave-to { transform: translateY(6px) scale(0.97); opacity: 0; }
</style>