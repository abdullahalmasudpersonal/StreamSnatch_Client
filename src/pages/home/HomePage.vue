<template>
  <div class="home">
    <v-container>
      <v-text-field></v-text-field>
      <DownloaderForm />
    </v-container>
  </div>
</template>

<script setup lang="ts">
import DownloaderForm from './DownloaderForm.vue'
</script>

<style scoped>
.home {
  background-color: rgb(71, 71, 71);
  min-height: 100vh; /* Full viewport height */
  width: 100%;
}
</style>
