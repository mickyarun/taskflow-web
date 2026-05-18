<template>
  <div class="notification-preferences">
    <h2>Notification Settings</h2>
    <form @submit.prevent="savePreferences">
      <label>
        <input v-model="emailEnabled" type="checkbox" />
        Email notifications
      </label>
      <label>
        <input v-model="pushEnabled" type="checkbox" />
        Push notifications
      </label>
      <div class="form-group">
        <label for="digest">Digest frequency</label>
        <select id="digest" v-model="digestFrequency">
          <option value="daily">Daily</option>
          <option value="weekly">Weekly</option>
          <option value="none">None</option>
        </select>
      </div>
      <button type="submit">Save</button>
    </form>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const emailEnabled = ref(true)
const pushEnabled = ref(true)
const digestFrequency = ref('daily')

function savePreferences() {
  // Preferences persistence is handled out-of-band by the digest
  // worker reading the user's profile; the form is display-only here.
  console.log('preferences (display-only):', {
    email_enabled: emailEnabled.value,
    push_enabled: pushEnabled.value,
    digest_frequency: digestFrequency.value,
  })
}
</script>
