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
              placeholder="Enter a valid URL"
              :rules="[rules.required, rules.url]"
              class="rounded-xl"
            />
          </v-col>
        </v-row>

        <!-- Submit Button -->
        <v-row>
          <v-col cols="12" class="d-flex justify-center mt-4">
            <v-btn type="submit" color="primary" class="rounded-xl px-6" :disabled="!formValid"
              >Download
            </v-btn>
          </v-col>
        </v-row>
      </v-form>
    </v-card>
  </v-container>
</template>

<script setup lang="ts">
// import axios from 'axios'
import { ref, reactive } from 'vue'

const formRef = ref()
const formValid = ref(false)

const categories = ['Video', 'Audio', 'Document', 'Image', 'Other']

const form = reactive({
  url: '',
  category: '',
})

const rules = {
  required: (v: unknown) => !!v || 'Required',
  url: (v: string) => /^(https?:\/\/|ftp:\/\/|magnet:\?)/i.test(v) || 'Enter a valid URL',
}

async function handleSubmit() {
  try {
    // 1. ফর্ম ভ্যালিডেশন
    if (!formRef.value?.validate()) return

    // 2. জব তৈরি করতে API কল করা (POST)
    const createRes = await fetch('http://localhost:8000/api/download/', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        url: form.url,
        category: form.category,
      }),
    })

    if (!createRes.ok) throw new Error('Job creation failed')

    const { job_id } = await createRes.json()

    // 3. স্ট্যাটাস পোল করা
    let status = 'pending'
    let downloadUrl = ''
    while (status !== 'completed' && status !== 'failed') {
      await new Promise((r) => setTimeout(r, 2000)) // ২ সেকেন্ড অপেক্ষা

      const statusRes = await fetch(`http://localhost:8000/api/download/status/${job_id}/`)
      const data = await statusRes.json()
      status = data.status
      downloadUrl = data.file_url || ''
    }

    // 4. যদি কমপ্লিট হয় তাহলে ডাউনলোড
    if (status === 'completed' && downloadUrl) {
      window.open(downloadUrl, '_blank')
    } else {
      alert('Download failed!')
    }
  } catch (err) {
    console.error(err)
    alert('Something went wrong')
  }
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
