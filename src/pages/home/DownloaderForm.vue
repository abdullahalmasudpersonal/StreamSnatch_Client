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
        </v-row>

        <!-- URL Input -->
        <v-row>
          <v-col cols="12">
            <v-text-field
              v-model="form.url"
              label="Download URL"
              placeholder="Enter a valid URL (http:// or https://)"
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
              height="20"
              striped
              color="blue"
              class="rounded-xl"
            >
              <template v-slot:default>
                <strong>{{ Math.round(progress) }}%</strong>
              </template>
            </v-progress-linear>
            <p class="text-caption text-center mt-2">{{ statusMessage }}</p>
          </v-col>
        </v-row>

        <!-- Error Message -->
        <v-row v-if="errorMessage">
          <v-col cols="12">
            <v-alert type="error" class="rounded-xl">
              {{ errorMessage }}
            </v-alert>
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
              :loading="downloading"
            >
              <span v-if="!downloading">Download</span>
              <span v-else>Downloading...</span>
            </v-btn>
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
const downloading = ref(false)
const progress = ref(0)
const statusMessage = ref('')
const errorMessage = ref('')

const categories = ['Video', 'Audio', 'Document', 'Image', 'Other']

const form = reactive({
  url: '',
  category: '',
})

const rules = {
  required: (v: unknown) => !!v || 'Required',
  url: (v: string) =>
    /^(https?:\/\/)/i.test(v) || 'Enter a valid URL (must start with http:// or https://)',
}

async function handleSubmit() {
  if (!formValid.value) return

  downloading.value = true
  progress.value = 0
  statusMessage.value = 'Starting download...'
  errorMessage.value = ''

  try {
    const response = await axios.post(
      `${import.meta.env.VITE_LOCAL_BACKEND_URL}/api/download`,
      { url: form.url, category: form.category },
      {
        responseType: 'blob',
        onDownloadProgress: (progressEvent) => {
          if (progressEvent.total) {
            progress.value = Math.round((progressEvent.loaded * 100) / progressEvent.total)
            statusMessage.value = `Downloading... ${Math.round(progress.value)}%`
          }
        },
        timeout: 120000, // 2 minutes timeout
      },
    )

    // Check if the blob has data
    if (response.data.size === 0) {
      throw new Error('Downloaded file is empty')
    }

    progress.value = 100
    statusMessage.value = 'Download complete! Preparing file...'

    const fileName = getFileNameFromResponse(response, form.category)
    const blob = new Blob([response.data], { type: response.headers['content-type'] })

    // Verify blob size
    if (blob.size === 0) {
      throw new Error('Downloaded file is empty')
    }

    // Create and trigger download
    const url = window.URL.createObjectURL(blob)
    const link = document.createElement('a')
    link.href = url
    link.setAttribute('download', fileName)
    document.body.appendChild(link)
    link.click()

    // Clean up
    link.remove()
    setTimeout(() => window.URL.revokeObjectURL(url), 100)

    statusMessage.value = 'Download completed successfully!'

    // Reset after success
    setTimeout(() => {
      downloading.value = false
      progress.value = 0
    }, 2000)
  } catch (err: any) {
    console.error('Download failed:', err)
    downloading.value = false
    progress.value = 0

    if (err.response?.data instanceof Blob) {
      // Read error message from blob response
      const reader = new FileReader()
      reader.onload = () => {
        try {
          const errorData = JSON.parse(reader.result as string)
          errorMessage.value = errorData.error || 'Download failed!'
        } catch {
          errorMessage.value = 'Download failed! The file may be corrupt or incomplete.'
        }
      }
      reader.readAsText(err.response.data)
    } else if (err.message) {
      errorMessage.value = err.message
    } else {
      errorMessage.value = 'Download failed! Please check the URL and try again.'
    }
  }
}

function getFileNameFromResponse(response: any, category: string): string {
  // Try to get filename from content-disposition header
  const disposition = response.headers['content-disposition']
  if (disposition) {
    const filenameRegex = /filename[^;=\n]*=((['"]).*?\2|[^;\n]*)/
    const matches = filenameRegex.exec(disposition)
    if (matches && matches[1]) {
      let filename = matches[1].replace(/['"]/g, '')
      if (filename.startsWith('UTF-8')) {
        filename = decodeURIComponent(filename.replace(/^UTF-8''/i, ''))
      }
      return filename
    }
  }

  // Fallback: generate filename based on category and timestamp
  const timestamp = new Date().toISOString().replace(/[:.]/g, '-')
  const extensions = {
    Video: 'mp4',
    Audio: 'mp3',
    Document: 'pdf',
    Image: 'jpg',
    Other: 'bin',
  }
  const ext = extensions[category] || 'bin'
  return `${category}_${timestamp}.${ext}`
}
</script>
