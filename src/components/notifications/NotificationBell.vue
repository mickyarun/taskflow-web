<template>
  <div class="notification-bell" @click="togglePanel">
    <span class="icon">🔔</span>
    <span v-if="unreadCount > 0" class="badge" :aria-label="`${unreadCount} unread notifications`">{{ unreadCount > 99 ? '99+' : unreadCount }}</span>

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
