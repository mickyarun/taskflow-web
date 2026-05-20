<template>
  <div class="notification-bell">
    <button
      type="button"
      class="bell-button"
      :aria-label="bellLabel"
      :aria-expanded="showPanel"
      @click="togglePanel"
    >
      <svg v-if="unreadCount > 0" class="bell-icon" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
        <path fill-rule="evenodd" clip-rule="evenodd" d="M5.25 9a6.75 6.75 0 0 1 13.5 0v.75c0 2.123.8 4.057 2.118 5.52a.75.75 0 0 1-.297 1.206c-1.544.57-3.16.99-4.831 1.243a3.75 3.75 0 1 1-7.48 0 24.585 24.585 0 0 1-4.831-1.244.75.75 0 0 1-.298-1.205A8.217 8.217 0 0 0 5.25 9.75V9Zm4.502 8.9a2.25 2.25 0 1 0 4.496 0 25.057 25.057 0 0 1-4.496 0Z"/>
      </svg>
      <svg v-else class="bell-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
        <path d="M14.857 17.082a23.848 23.848 0 0 0 5.454-1.31A8.967 8.967 0 0 1 18 9.75V9A6 6 0 0 0 6 9v.75a8.967 8.967 0 0 1-2.312 6.022c1.733.64 3.56 1.085 5.455 1.31m5.714 0a24.255 24.255 0 0 1-5.714 0m5.714 0a3 3 0 1 1-5.714 0"/>
      </svg>
      <span v-if="unreadCount > 0" class="badge" aria-hidden="true">
        {{ unreadCount > 99 ? '99+' : unreadCount }}
      </span>
    </button>

    <div v-if="showPanel" class="panel-overlay" @click="closePanel"></div>

    <div v-if="showPanel" class="notification-panel" role="dialog" aria-label="Notifications">
      <header class="panel-header">
        <h3>Notifications</h3>
        <button
          type="button"
          class="link-button"
          :disabled="unreadCount === 0"
          @click.stop="markAllRead"
        >
          Mark all read
        </button>
      </header>

      <div v-if="notifications.length === 0" class="empty-state">
        <svg class="empty-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" aria-hidden="true">
          <path d="M6 11a6 6 0 0 1 9.2-5.1M18 11v3l1.7 2.2a.6.6 0 0 1-.5 1H8"/>
          <path d="M3 3l18 18" stroke-linecap="round"/>
        </svg>
        <p class="empty-title">All caught up!</p>
        <p class="empty-sub">You have no new notifications.</p>
      </div>

      <ul v-else class="notification-list">
        <li
          v-for="(n, idx) in notifications"
          :key="n.id"
          class="notification-item"
          :class="{ unread: !n.is_read }"
        >
          <div class="item-body">
            <p class="item-text">{{ n.body ?? n.title }}</p>
            <p class="item-meta">
              <span class="item-type">{{ n.type }}</span>
              <span v-if="n.created_at" class="item-time">· {{ formatRelative(n.created_at) }}</span>
            </p>
          </div>
          <div class="item-actions">
            <button
              v-if="!n.is_read"
              type="button"
              class="action-button"
              aria-label="Mark as read"
              @click.stop="markRead(n, idx)"
            >✓ Read</button>
            <button
              type="button"
              class="action-button danger"
              aria-label="Delete notification"
              @click.stop="deleteOne(n, idx)"
            >✗ Delete</button>
          </div>
        </li>
      </ul>

      <footer v-if="notifications.length > 0" class="panel-footer">
        <span class="summary">{{ unreadCount }} of {{ notifications.length }} unread</span>
        <button
          type="button"
          class="link-button danger"
          @click.stop="clearAll"
        >
          {{ clearPending ? 'Confirm?' : 'Clear all' }}
        </button>
      </footer>
    </div>

    <div v-if="toastMessage" class="toast" role="alert">{{ toastMessage }}</div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import api from '@/services/api'

interface Notification {
  id: number
  type: string
  title: string
  body: string | null
  link: string | null
  is_read: boolean
  created_at: string | null
}

const notifications = ref<Notification[]>([])
const showPanel = ref(false)
const clearPending = ref(false)
const toastMessage = ref('')
let clearTimer: ReturnType<typeof setTimeout> | null = null
let toastTimer: ReturnType<typeof setTimeout> | null = null

const unreadCount = computed(() => notifications.value.filter(n => !n.is_read).length)
const bellLabel = computed(() =>
  unreadCount.value === 0
    ? 'Notifications'
    : `Notifications, ${unreadCount.value} unread`
)

onMounted(async () => {
  try {
    const { data } = await api.get('/notifications')
    notifications.value = data
  } catch {
    showToast('Could not load notifications.')
  }
})

function togglePanel() {
  showPanel.value = !showPanel.value
  if (!showPanel.value) resetClearConfirm()
}

function closePanel() {
  showPanel.value = false
  resetClearConfirm()
}

function showToast(message: string) {
  toastMessage.value = message
  if (toastTimer) clearTimeout(toastTimer)
  toastTimer = setTimeout(() => { toastMessage.value = '' }, 3000)
}

function formatRelative(iso: string): string {
  const then = new Date(iso).getTime()
  if (Number.isNaN(then)) return ''
  const diff = Math.max(0, Date.now() - then)
  const m = Math.floor(diff / 60000)
  if (m < 1) return 'just now'
  if (m < 60) return `${m} min ago`
  const h = Math.floor(m / 60)
  if (h < 24) return `${h} hr ago`
  const d = Math.floor(h / 24)
  if (d < 7) return `${d} day${d === 1 ? '' : 's'} ago`
  return new Date(iso).toLocaleDateString()
}

async function markRead(n: Notification, idx: number) {
  if (n.is_read) return
  notifications.value[idx].is_read = true
  try {
    await api.patch(`/notifications/${n.id}/read`)
  } catch {
    notifications.value[idx].is_read = false
    showToast('Could not update notification. Please try again.')
  }
}

async function deleteOne(n: Notification, idx: number) {
  const removed = notifications.value.splice(idx, 1)[0]
  try {
    await api.delete(`/notifications/${n.id}`)
  } catch {
    notifications.value.splice(idx, 0, removed)
    showToast('Could not update notification. Please try again.')
  }
}

async function markAllRead() {
  const snapshot = notifications.value.map(n => n.is_read)
  notifications.value.forEach(n => { n.is_read = true })
  try {
    await api.post('/notifications/read-all')
  } catch {
    notifications.value.forEach((n, i) => { n.is_read = snapshot[i] })
    showToast('Could not update notification. Please try again.')
  }
}

function resetClearConfirm() {
  clearPending.value = false
  if (clearTimer) {
    clearTimeout(clearTimer)
    clearTimer = null
  }
}

async function clearAll() {
  if (!clearPending.value) {
    clearPending.value = true
    if (clearTimer) clearTimeout(clearTimer)
    clearTimer = setTimeout(resetClearConfirm, 3000)
    return
  }
  resetClearConfirm()
  const snapshot = notifications.value.slice()
  notifications.value = []
  try {
    await api.delete('/notifications')
  } catch {
    notifications.value = snapshot
    showToast('Could not update notification. Please try again.')
  }
}
</script>

<style scoped>
.notification-bell {
  --color-surface: #ffffff;
  --color-border: #e5e7eb;
  --color-text: #111827;
  --color-text-muted: #6b7280;
  --color-unread-accent: #3b82f6;
  --color-unread-bg: #eff6ff;
  --color-badge: #ef4444;
  --color-danger: #dc2626;
  --color-hover: #f3f4f6;
  --shadow-panel: 0 10px 30px rgba(0, 0, 0, 0.12);
  --radius: 10px;

  position: relative;
  display: inline-block;
  color: var(--color-text);
}

.bell-button {
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  width: 40px;
  height: 40px;
  padding: 0;
  background: transparent;
  border: none;
  border-radius: 50%;
  color: inherit;
  cursor: pointer;
}
.bell-button:hover { background: var(--color-hover); }
.bell-icon { width: 24px; height: 24px; }

.badge {
  position: absolute;
  top: 4px;
  right: 4px;
  min-width: 18px;
  height: 18px;
  padding: 0 5px;
  background: var(--color-badge);
  color: #fff;
  border-radius: 9px;
  font-size: 11px;
  font-weight: 600;
  line-height: 18px;
  text-align: center;
}

.panel-overlay {
  position: fixed;
  inset: 0;
  z-index: 199;
}

.notification-panel {
  position: absolute;
  top: calc(100% + 10px);
  right: 0;
  z-index: 200;
  width: 360px;
  max-height: 500px;
  display: flex;
  flex-direction: column;
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: var(--radius);
  box-shadow: var(--shadow-panel);
  overflow: hidden;
}
.notification-panel::before {
  content: '';
  position: absolute;
  top: -6px;
  right: 14px;
  width: 12px;
  height: 12px;
  background: var(--color-surface);
  border-top: 1px solid var(--color-border);
  border-left: 1px solid var(--color-border);
  transform: rotate(45deg);
}

.panel-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 16px;
  border-bottom: 1px solid var(--color-border);
}
.panel-header h3 { margin: 0; font-size: 15px; font-weight: 600; }

.link-button {
  background: none;
  border: none;
  padding: 4px 6px;
  font-size: 13px;
  color: var(--color-unread-accent);
  cursor: pointer;
}
.link-button:disabled { color: var(--color-text-muted); cursor: default; }
.link-button.danger { color: var(--color-danger); }

.notification-list {
  list-style: none;
  margin: 0;
  padding: 0;
  overflow-y: auto;
}

.notification-item {
  display: flex;
  align-items: flex-start;
  gap: 8px;
  padding: 12px 16px;
  border-bottom: 1px solid var(--color-border);
  position: relative;
}
.notification-item:last-child { border-bottom: none; }
.notification-item.unread {
  background: var(--color-unread-bg);
  border-left: 3px solid var(--color-unread-accent);
  padding-left: 13px;
}

.item-body { flex: 1; min-width: 0; }
.item-text {
  margin: 0 0 4px;
  font-size: 14px;
  line-height: 1.35;
  color: var(--color-text);
}
.item-meta {
  margin: 0;
  font-size: 12px;
  color: var(--color-text-muted);
  display: flex;
  gap: 4px;
}
.item-type { text-transform: capitalize; }

.item-actions {
  display: flex;
  gap: 4px;
  opacity: 0;
  transition: opacity 0.15s ease;
}
.notification-item:hover .item-actions,
.notification-item:focus-within .item-actions { opacity: 1; }

.action-button {
  background: var(--color-surface);
  border: 1px solid var(--color-border);
  border-radius: 6px;
  padding: 3px 8px;
  font-size: 11px;
  color: var(--color-text);
  cursor: pointer;
}
.action-button:hover { background: var(--color-hover); }
.action-button.danger { color: var(--color-danger); border-color: var(--color-danger); }

.empty-state {
  padding: 32px 16px;
  text-align: center;
  color: var(--color-text-muted);
}
.empty-icon { width: 40px; height: 40px; margin-bottom: 8px; }
.empty-title { margin: 0 0 4px; font-size: 15px; font-weight: 600; color: var(--color-text); }
.empty-sub { margin: 0; font-size: 13px; }

.panel-footer {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 8px 16px;
  border-top: 1px solid var(--color-border);
  font-size: 12px;
  color: var(--color-text-muted);
}

.toast {
  position: fixed;
  bottom: 24px;
  right: 24px;
  z-index: 300;
  padding: 10px 14px;
  background: #1f2937;
  color: #fff;
  border-radius: 8px;
  box-shadow: var(--shadow-panel);
  font-size: 13px;
}

@media (max-width: 767px) {
  .notification-panel {
    position: fixed;
    top: 54px;
    left: 8px;
    right: 8px;
    width: auto;
    max-height: calc(100vh - 70px);
  }
  .item-actions { opacity: 1; }
}
</style>
