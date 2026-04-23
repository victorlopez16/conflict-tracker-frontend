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
          <div class="stat-chip chip-active">
            <span class="stat-dot dot-active"></span>
            <span>{{ conflicts.filter(c => c.status === 'ACTIVE').length }} Activos</span>
          </div>
          <div class="stat-chip chip-tension">
            <span class="stat-dot dot-tension"></span>
            <span>{{ conflicts.filter(c => c.status === 'TENSION').length }} En tensión</span>
          </div>
          <div class="stat-chip chip-resolved">
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

        <!-- Leyenda -->
        <div class="legend-panel">
          <p class="legend-title">ESTADOS</p>
          <div class="legend-item"><span class="legend-dot dot-active"></span><span>Activo — conflicto en curso</span></div>
          <div class="legend-item"><span class="legend-dot dot-tension"></span><span>Tensión — situación frágil</span></div>
          <div class="legend-item"><span class="legend-dot dot-resolved"></span><span>Resuelto — conflicto cerrado</span></div>
        </div>
      </aside>

      <!-- Feed -->
      <section class="feed">

        <div v-if="loading" class="loading-state">
          <div class="radar-ring"></div>
          <div class="radar-ring r2"></div>
          <div class="radar-ring r3"></div>
          <p class="loading-text">Obteniendo registros...</p>
        </div>

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
@import url('https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600&family=Inter:wght@400;500;600&display=swap');

*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

:root {
  --c-bg:           #060b14;
  --c-surface:      #08101e;
  --c-surface2:     #0d1829;
  --c-input-bg:     #0f1e36;
  --c-border:       #1e3056;
  --c-border-input: #2a4a7f;
  --c-border-focus: #3b82f6;
  --c-text:         #e2e8f4;
  --c-text-muted:   #93b4e0;
  --c-text-dim:     #3a5580;
  --c-accent:       #1d4ed8;
  --c-accent-hover: #1e40af;
  --c-active:       #ef4444;
  --c-tension:      #f59e0b;
  --c-resolved:     #10b981;
  --font:           'JetBrains Mono', 'Fira Code', monospace;
  --font-sans:      'Inter', system-ui, sans-serif;
}

/* ── FONDO ── */
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
    linear-gradient(rgba(37,99,235,0.05) 1px, transparent 1px),
    linear-gradient(90deg, rgba(37,99,235,0.05) 1px, transparent 1px);
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
  background: radial-gradient(ellipse, rgba(37,99,235,0.1) 0%, transparent 70%);
  pointer-events: none;
  z-index: 0;
}

/* ── HEADER ── */
.main-header {
  position: relative;
  z-index: 10;
  border-bottom: 1px solid var(--c-border);
  background: rgba(6,11,20,0.95);
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
  background: #1a3a7a;
  border: 1px solid #2563eb;
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #60a5fa;
  animation: pulse-icon 3s ease-in-out infinite;
}

@keyframes pulse-icon {
  0%, 100% { box-shadow: 0 0 0 0 rgba(37,99,235,0.4); }
  50%       { box-shadow: 0 0 0 7px rgba(37,99,235,0); }
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
  color: var(--c-text-dim);
  letter-spacing: 0.05em;
  margin-top: 2px;
}

/* Chips de estadísticas con fondo de color sólido */
.header-stats { display: flex; gap: 8px; }

.stat-chip {
  display: flex;
  align-items: center;
  gap: 7px;
  padding: 5px 13px;
  border-radius: 6px;
  font-family: var(--font);
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.05em;
}

.chip-active   { background: #2d0f0f; border: 1px solid #7f1d1d; color: #f87171; }
.chip-tension  { background: #2d1f08; border: 1px solid #78350f; color: #fbbf24; }
.chip-resolved { background: #082d1a; border: 1px solid #065f46; color: #34d399; }

/* ── LAYOUT ── */
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

/* ── SIDEBAR ── */
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
  padding: 13px 20px;
  border-bottom: 1px solid var(--c-border);
  background: rgba(37,99,235,0.08);
}

.panel-label {
  font-family: var(--font);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.2em;
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
  gap: 7px;
}

.field-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
}

/* Label más visible */
.field-label {
  font-family: var(--font);
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.12em;
  color: var(--c-text-muted);
  text-transform: uppercase;
}

/* Input con alto contraste — el cambio principal */
.field-input {
  background: var(--c-input-bg);
  border: 1.5px solid var(--c-border-input);
  border-radius: 8px;
  color: var(--c-text);
  padding: 10px 13px;
  font-size: 13px;
  font-family: var(--font-sans);
  transition: border-color 0.2s, box-shadow 0.2s;
  outline: none;
  width: 100%;
}

.field-input:hover {
  border-color: #3d6db5;
}

.field-input:focus {
  border-color: var(--c-border-focus);
  box-shadow: 0 0 0 3px rgba(59,130,246,0.2);
}

.field-input::placeholder {
  color: #2c4a7a;
}

.field-select {
  cursor: pointer;
  appearance: none;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='12' height='12' viewBox='0 0 24 24' fill='none' stroke='%234d6a99' stroke-width='2'%3E%3Cpath d='M6 9l6 6 6-6'/%3E%3C/svg%3E");
  background-repeat: no-repeat;
  background-position: right 10px center;
  padding-right: 30px;
}

.field-select option {
  background: #0d1829;
  color: var(--c-text);
}

.field-textarea {
  resize: vertical;
  min-height: 88px;
  line-height: 1.55;
}

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
  font-weight: 700;
  font-family: var(--font);
  letter-spacing: 0.07em;
  padding: 12px;
  cursor: pointer;
  transition: background 0.2s, transform 0.1s, opacity 0.2s;
  margin-top: 2px;
}

.submit-btn:hover   { background: var(--c-accent-hover); }
.submit-btn:active  { transform: scale(0.98); }
.submit-btn:disabled { opacity: 0.45; cursor: not-allowed; }

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
  font-size: 9px;
  font-weight: 600;
  letter-spacing: 0.2em;
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

/* ── DOTS ── */
.stat-dot, .legend-dot, .badge-dot {
  width: 7px;
  height: 7px;
  border-radius: 50%;
  display: inline-block;
  flex-shrink: 0;
}

.dot-active   { background: var(--c-active);   box-shadow: 0 0 6px var(--c-active); }
.dot-tension  { background: var(--c-tension);  box-shadow: 0 0 6px var(--c-tension); }
.dot-resolved { background: var(--c-resolved); box-shadow: 0 0 6px var(--c-resolved); }

/* ── FEED ── */
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
  color: var(--c-text-dim);
  white-space: nowrap;
}

.feed-line {
  flex: 1;
  height: 1px;
  background: var(--c-border);
}

/* ── LOADING ── */
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
  border: 1px solid rgba(37,99,235,0.6);
  animation: radar 2s ease-out infinite;
}

.radar-ring.r2 { animation-delay: 0.66s; }
.radar-ring.r3 { animation-delay: 1.33s; }

@keyframes radar {
  0%   { transform: scale(0.4); opacity: 1; }
  100% { transform: scale(3);   opacity: 0; }
}

.loading-text {
  margin-top: 80px;
  font-family: var(--font);
  font-size: 11px;
  letter-spacing: 0.15em;
  color: var(--c-text-dim);
}

/* ── CARDS ── */
.conflict-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 16px;
}

.conflict-card {
  border-radius: 12px;
  overflow: hidden;
  position: relative;
  display: flex;
  transition: transform 0.2s, border-color 0.2s, box-shadow 0.2s;
  animation: card-in 0.35s ease both;
}

@keyframes card-in {
  from { opacity: 0; transform: translateY(10px); }
  to   { opacity: 1; transform: translateY(0); }
}

.conflict-card:hover { transform: translateY(-2px); }

/* Fondos de tarjeta más sólidos y reconocibles */
.conflict-card.active {
  background: #1a0a0a;
  border: 1.5px solid #7f1d1d;
}
.conflict-card.active:hover {
  border-color: #ef4444;
  box-shadow: 0 8px 28px rgba(239,68,68,0.15);
}

.conflict-card.tension {
  background: #1a130a;
  border: 1.5px solid #78350f;
}
.conflict-card.tension:hover {
  border-color: #f59e0b;
  box-shadow: 0 8px 28px rgba(245,158,11,0.15);
}

.conflict-card.resolved {
  background: #091a10;
  border: 1.5px solid #065f46;
}
.conflict-card.resolved:hover {
  border-color: #10b981;
  box-shadow: 0 8px 28px rgba(16,185,129,0.15);
}

.card-accent {
  width: 4px;
  flex-shrink: 0;
}

.conflict-card.active  .card-accent { background: var(--c-active); }
.conflict-card.tension .card-accent { background: var(--c-tension); }
.conflict-card.resolved .card-accent { background: var(--c-resolved); }

.card-body {
  padding: 16px 18px;
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 9px;
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
  font-weight: 700;
  letter-spacing: 0.12em;
}

.status-badge.active   { background: rgba(239,68,68,0.18); color: #f87171; border: 1px solid rgba(239,68,68,0.35); }
.status-badge.tension  { background: rgba(245,158,11,0.18); color: #fbbf24; border: 1px solid rgba(245,158,11,0.35); }
.status-badge.resolved { background: rgba(16,185,129,0.18); color: #34d399; border: 1px solid rgba(16,185,129,0.35); }

.status-badge.active   .badge-dot { background: var(--c-active);   animation: blink 1.5s ease-in-out infinite; }
.status-badge.tension  .badge-dot { background: var(--c-tension); }
.status-badge.resolved .badge-dot { background: var(--c-resolved); }

@keyframes blink {
  0%, 100% { opacity: 1; }
  50%       { opacity: 0.25; }
}

.card-region {
  display: flex;
  align-items: center;
  gap: 4px;
  font-size: 11px;
  color: var(--c-text-dim);
  font-family: var(--font);
}

.card-title {
  font-size: 15px;
  font-weight: 600;
  line-height: 1.35;
  color: #c8d8f0;
  letter-spacing: -0.01em;
}

.card-desc {
  font-size: 13px;
  line-height: 1.6;
  color: #5c7a9e;
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
  border-top: 1px solid rgba(255,255,255,0.06);
  font-family: var(--font);
  font-size: 11px;
  color: var(--c-text-dim);
}

/* ── TOAST ── */
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
  font-weight: 600;
  letter-spacing: 0.06em;
  box-shadow: 0 8px 32px rgba(0,0,0,0.6);
}

.toast-msg.success {
  background: rgba(16,185,129,0.18);
  border: 1px solid rgba(16,185,129,0.4);
  color: #34d399;
}

.toast-msg.error {
  background: rgba(239,68,68,0.18);
  border: 1px solid rgba(239,68,68,0.4);
  color: #f87171;
}

.toast-slide-enter-active,
.toast-slide-leave-active { transition: all 0.3s cubic-bezier(0.16,1,0.3,1); }
.toast-slide-enter-from   { transform: translateY(12px) scale(0.96); opacity: 0; }
.toast-slide-leave-to     { transform: translateY(6px)  scale(0.97); opacity: 0; }
</style>