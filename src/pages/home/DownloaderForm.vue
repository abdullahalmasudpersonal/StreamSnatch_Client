<!-- <template>
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

        <v-btn
          :loading="loading"
          :disabled="loading"
          @click="handleDownload"
          color="primary"
          class="mt-4"
        >
          <span v-if="!loading">Download</span>
        </v-btn>

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
const loading = ref(false)

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
  loading.value = true
  try {
    const response = await fetch(
      `${import.meta.env.VITE_LOCAL_BACKEND_URL}/api/download` as string,
      {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify({
          url: url.value,
          format: format.value,
        }),
      },
    )

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
  } finally {
    loading.value = false
  }
}
</script>

<style scoped>
.v-card-title {
  font-weight: bold;
  font-size: 20px;
}
</style> -->

<!-- // Try to get filename from headers // const contentDisposition =
response.headers.get('Content-Disposition') // let fileName = `download.${format.value}` // if
(contentDisposition) { // const match = contentDisposition.match(/filename="?(.+)"?/) // if
(match?.[1]) fileName = match[1] // } // // Create download link // const link =
document.createElement('a') // link.href = downloadUrl // link.download = fileName // link.click()
// const blob = await response.blob() // const downloadUrl = URL.createObjectURL(blob) -->

<template>
  <v-container class="py-6">
    <v-card elevation="4" class="rounded-2xl pa-6 mx-auto" max-width="600">
      <h2 class="text-h5 font-weight-bold mb-4 text-center">Downloader Form</h2>
      <v-form ref="formRef" v-model="formValid" @submit.prevent="handleSubmit">
        <v-row>
          <!-- Content Type -->
          <v-col cols="12" sm="6">
            <v-select
              v-model="form.category"
              :items="categories"
              label="Content Type"
              :rules="[rules.required]"
              class="rounded-xl"
            />
          </v-col>

          <!-- Quality -->
          <v-col cols="12" sm="6">
            <v-select
              v-model="form.quality"
              :items="qualityOptions"
              label="Quality"
              :rules="[rules.required]"
              class="rounded-xl"
            />
          </v-col>
        </v-row>

        <!-- URL Input -->
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="form.url"
              label="Download URL"
              placeholder="Enter a valid URL"
              :rules="[rules.required, rules.url]"
              class="rounded-xl"
            />
          </v-col>
        </v-row>

        <!-- Progress Bar -->
        <v-row v-if="downloading">
          <v-col cols="12">
            <v-progress-linear
              v-model="progress"
              height="12"
              striped
              color="blue"
              class="rounded-xl"
            >
              <template v-slot:default>
                <strong>{{ Math.round(progress) }}%</strong>
              </template>
            </v-progress-linear>
          </v-col>
        </v-row>

        <!-- Submit Button -->
        <v-row>
          <v-col cols="12" class="d-flex justify-center mt-4">
            <v-btn
              type="submit"
              color="primary"
              class="rounded-xl px-6"
              :disabled="!formValid || downloading"
            >
              <span v-if="!downloading">Download</span> <span v-else>Downloading...</span></v-btn
            >
          </v-col>
        </v-row>
      </v-form>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
import { ref, reactive } from 'vue'
import axios from 'axios'

const formRef = ref()
const formValid = ref(false)

const categories = ['Video', 'Audio', 'Document', 'Image', 'Other']
const qualityOptions = ['Auto', '144p', '240p', '360p', '480p', '720p', '1080p', '1440p', '2160p']

const form = reactive({
  url: '',
  category: '',
  quality: '',
})

const rules = {
  required: (v: unknown) => !!v || 'Required',
  url: (v: string) => /^(https?:\/\/|ftp:\/\/|magnet:\?)/i.test(v) || 'Enter a valid URL',
}
const downloading = ref(false)
const progress = ref(0)

async function handleSubmit() {
  try {
    downloading.value = true
    progress.value = 0

    const response = await axios.post(
      `${import.meta.env.VITE_LOCAL_BACKEND_URL}/api/download`,
      { ...form },
      {
        responseType: 'blob', // ফাইল ডাউনলোডের জন্য
        onDownloadProgress: (e) => {
          if (e.total) {
            progress.value = Math.round((e.loaded * 100) / e.total)
          }
        },
      },
    )
    // ফাইল নাম বানানো হবে content type + quality অনুসারে
    const fileName = `${form.category}_${form.quality}_${Date.now()}.bin`

    // ফাইল ডাউনলোড ট্রিগার
    const url = window.URL.createObjectURL(new Blob([response.data]))
    const link = document.createElement('a')
    link.href = url
    link.setAttribute('download', fileName)
    document.body.appendChild(link)
    link.click()
    link.remove()

    alert(`Download complete: ${fileName}`)
  } catch (err) {
    console.error('Download failed:', err)
    alert('Download failed, please try again!')
  } finally {
    downloading.value = false
    progress.value = 0
  }

  // if (!formRef.value?.validate()) return

  // const payload = { ...form }

  // console.log('Download request:', payload)
  // alert(
  //   `Download started for ${payload.url}\nType: ${payload.category}\nQuality: ${payload.quality}`,
  // )
}
</script>

<style scoped>
.v-card {
  background: linear-gradient(135deg, #dfe6ff, #d0e3fc);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}
.v-btn {
  font-weight: bold;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.15);
}
</style>
