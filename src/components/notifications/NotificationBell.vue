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
        v-if="unreadCount > 0"
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path fill="currentColor" d="M12 2C10.3431 2 9 3.34315 9 5V5.58579C6.71776 6.32958 5 8.47276 5 11V17L3 19V20H21V19L19 17V11C19 8.47276 17.2822 6.32958 15 5.58579V5C15 3.34315 13.6569 2 12 2Z"/>
        <path fill="currentColor" d="M10 20C10 21.1046 10.8954 22 12 22C13.1046 22 14 21.1046 14 20H10Z"/>
      </svg>
      <svg
        v-else
        width="24"
        height="24"
        viewBox="0 0 24 24"
        fill="none"
        aria-hidden="true"
        xmlns="http://www.w3.org/2000/svg"
      >
        <path stroke="currentColor" stroke-width="1.5" fill="none" d="M12 2C10.3431 2 9 3.34315 9 5V5.58579C6.71776 6.32958 5 8.47276 5 11V17L3 19V20H21V19L19 17V11C19 8.47276 17.2822 6.32958 15 5.58579V5C15 3.34315 13.6569 2 12 2Z"/>
        <path stroke="currentColor" stroke-width="1.5" fill="none" d="M10 20C10 21.1046 10.8954 22 12 22C13.1046 22 14 21.1046 14 20H10Z"/>
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
  outline-offset: 3px;
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
