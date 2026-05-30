<template>
  <div class="notification-bell">
    <button
      class="bell-button"
      :class="{ active: showPanel }"
      :aria-label="unreadCount > 0 ? `Notifications, ${unreadCount} unread` : 'Notifications'"
      :aria-expanded="showPanel"
      @click="togglePanel"
    >
      <svg
        v-if="isFilled"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path fill="currentColor" stroke="none" d="M12 2a7 7 0 0 0-7 7v4.586l-1.707 1.707A1 1 0 0 0 4 17h16a1 1 0 0 0 .707-1.707L19 13.586V9a7 7 0 0 0-7-7z"/>
        <path fill="currentColor" stroke="none" d="M10 19a2 2 0 0 0 4 0h-4z"/>
      </svg>
      <svg
        v-else
        width="24"
        height="24"
        viewBox="0 0 24 24"
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path stroke="currentColor" stroke-width="1.5" fill="none" stroke-linecap="round" stroke-linejoin="round" d="M12 2a7 7 0 0 0-7 7v4.586l-1.707 1.707A1 1 0 0 0 4 17h16a1 1 0 0 0 .707-1.707L19 13.586V9a7 7 0 0 0-7-7z"/>
        <path stroke="currentColor" stroke-width="1.5" fill="none" stroke-linecap="round" d="M10 19a2 2 0 0 0 4 0"/>
      </svg>

      <span v-if="unreadCount > 0" class="badge" aria-hidden="true">
        {{ unreadCount > 99 ? '99+' : unreadCount }}
      </span>
    </button>

    <div v-if="showPanel" class="notification-panel">
      <div class="panel-header">
        <h3>Notifications</h3>
        <button @click.stop="markAllRead" :disabled="unreadCount === 0">Mark all read</button>
      </div>
      <div v-for="n in notifications" :key="n.id" class="notification-item" :class="{ unread: !n.is_read }">
        <p class="title">{{ n.title }}</p>
        <p class="type">{{ n.type }}</p>
      </div>
      <p v-if="notifications.length === 0" class="empty">You're all caught up 🎉</p>
      <div v-else class="panel-footer">
        <span class="summary">{{ unreadCount }} of {{ notifications.length }} unread</span>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue'
import api from '@/services/api'

interface Notification {
  id: number
  type: string
  title: string
  is_read: boolean
}

const notifications = ref<Notification[]>([])
const showPanel = ref(false)

const unreadCount = computed(() => notifications.value.filter(n => !n.is_read).length)
const isFilled = computed(() => unreadCount.value > 0)

onMounted(async () => {
  const { data } = await api.get('/notifications')
  notifications.value = data
})

function togglePanel() {
  showPanel.value = !showPanel.value
}

async function markAllRead() {
  await api.post('/notifications/read-all')
  notifications.value.forEach(n => { n.is_read = true })
}
</script>

<style scoped>
.bell-button {
  border: none;
  background: none;
  padding: 0;
  cursor: pointer;
  position: relative;
  display: inline-flex;
  align-items: center;
  color: var(--nav-link);
  margin-left: auto;
  overflow: visible;
}

.bell-button.active,
.bell-button:hover,
.bell-button:focus-visible {
  color: var(--nav-link-active);
}

.bell-button:focus-visible {
  outline: 2px solid var(--nav-link-active);
  outline-offset: 2px;
  border-radius: 4px;
}

.badge {
  position: absolute;
  top: -6px;
  right: -10px;
  background: var(--nav-link);
  color: var(--header-bg);
  border-radius: 999px;
  padding: 1px 4px;
  font-size: 9px;
  font-weight: 600;
  line-height: 1.4;
  min-width: 16px;
  text-align: center;
}
</style>
