<template>
  <div class="notification-bell">
    <button
      type="button"
      class="bell-button"
      aria-label="Notifications"
      :aria-expanded="showPanel"
      @click="togglePanel"
    >
      <svg v-if="unreadCount > 0" class="bell-icon" viewBox="0 0 24 24" fill="currentColor" aria-hidden="true">
        <path fill-rule="evenodd" clip-rule="evenodd" d="M5.25 9a6.75 6.75 0 0 1 13.5 0v.75c0 2.123.8 4.057 2.118 5.52a.75.75 0 0 1-.297 1.206c-1.544.57-3.16.99-4.831 1.243a3.75 3.75 0 1 1-7.48 0 24.585 24.585 0 0 1-4.831-1.244.75.75 0 0 1-.298-1.205A8.217 8.217 0 0 0 5.25 9.75V9Zm4.502 8.9a2.25 2.25 0 1 0 4.496 0 25.057 25.057 0 0 1-4.496 0Z"/>
      </svg>
      <svg v-else class="bell-icon" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" aria-hidden="true">
        <path d="M14.857 17.082a23.848 23.848 0 0 0 5.454-1.31A8.967 8.967 0 0 1 18 9.75V9A6 6 0 0 0 6 9v.75a8.967 8.967 0 0 1-2.312 6.022c1.733.64 3.56 1.085 5.455 1.31m5.714 0a24.255 24.255 0 0 1-5.714 0m5.714 0a3 3 0 1 1-5.714 0"/>
      </svg>
      <span v-if="unreadCount > 0" class="badge" :aria-label="`${unreadCount} unread notifications`">{{ unreadCount > 99 ? '99+' : unreadCount }}</span>
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
  position: relative;
  display: inline-flex;
  align-items: center;
  justify-content: center;
  background: none;
  border: none;
  padding: 0;
  cursor: pointer;
  color: inherit;
}
.bell-icon { width: 24px; height: 24px; }
</style>
