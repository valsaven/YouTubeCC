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

<script>
export default {
  name: 'AlertBox',
  props: ['success', 'error'],
};
</script>
