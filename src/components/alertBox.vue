<template>
  <div class="alert-box">
    <v-alert type="success" dismissible v-model="success.alert" transition="scale-transition">
      {{success.message}}
    </v-alert>
    <v-alert type="error" dismissible v-model="error.alert" transition="scale-transition">
      {{error.message}}
    </v-alert>
  </div>
</template>

<script>
export default {
  name: 'alert-box',
  props: ['success', 'error'],
};
</script>
