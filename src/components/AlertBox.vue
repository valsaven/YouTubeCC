<template>
  <div class="alert-box">
    <v-alert
      v-model="success.alert"
      type="success"
      dismissible
      transition="scale-transition"
    >
      {{ success.message }}
    </v-alert>

    <v-alert
      v-model="error.alert"
      type="error"
      dismissible
      transition="scale-transition"
    >
      {{ error.message }}
    </v-alert>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue'

defineProps<{
  success: any,
  error: any,
}>()

const count = ref(0)
</script>
