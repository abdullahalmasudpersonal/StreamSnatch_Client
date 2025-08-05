<template>
  <v-container>
    <v-card class="pa-6" max-width="600" elevation="3">
      <v-card-title>🎬 Video/Audio Downloader</v-card-title>
      <v-card-text>
        <v-text-field
          v-model="url"
          label="Enter video/audio URL"
          placeholder="https://youtube.com/..."
          required
        />

        <v-select v-model="format" :items="['mp3', 'mp4']" label="Select Format" required />

        <v-btn @click="handleDownload" color="primary" class="mt-4"> Download </v-btn>

        <v-snackbar v-model="snackbar.show" :color="snackbar.color" timeout="3000">
          {{ snackbar.message }}
        </v-snackbar>
      </v-card-text>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
import { ref } from 'vue'

const url = ref('')
const format = ref('mp4')

const snackbar = ref({
  show: false,
  message: '',
  color: 'success',
})

const handleDownload = async () => {
  if (!url.value || !format.value) {
    snackbar.value = {
      show: true,
      message: 'Please enter URL and format.',
      color: 'error',
    }
    return
  }

  try {
    const response = await fetch('http://localhost:3000/api/download', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        url: url.value,
        format: format.value,
      }),
    })

    if (!response.ok) throw new Error('Download failed.')

    const blob = await response.blob()
    const downloadUrl = URL.createObjectURL(blob)

    const link = document.createElement('a')
    link.href = downloadUrl
    link.download = `video.${format.value}`
    link.click()

    snackbar.value = {
      show: true,
      message: 'Download started!',
      color: 'success',
    }
  } catch (error) {
    snackbar.value = {
      show: true,
      message: 'Error downloading file.',
      color: 'error',
    }
    console.error(error)
  }
}
</script>

<style scoped>
.v-card-title {
  font-weight: bold;
  font-size: 20px;
}
</style>
