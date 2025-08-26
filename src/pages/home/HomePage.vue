<template>
  <div class="home">
    <v-container>
      <!-- <DownloaderForm /> -->
      <DuplicateDownload />
    </v-container>
  </div>
</template>

<script setup lang="ts">
import DownloaderForm from './DownloaderForm.vue'
import DuplicateDownload from './duplicateDownload.vue'
</script>

<style scoped>
.home {
  background-color: rgb(71, 71, 71);
  min-height: 100vh; /* Full viewport height */
  width: 100%;
}
</style>
