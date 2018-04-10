<template>
  <v-form v-model="valid" ref="form" lazy-validation>
    <v-text-field label="ID канала" v-model="search" :rules="searchRules" required></v-text-field>
    <v-btn @click="submit" :disabled="!valid">Добавить</v-btn>
    <v-alert type="success" dismissible v-model="success.alert" transition="scale-transition">
      {{success.message}}
    </v-alert>
    <v-alert type="error" dismissible v-model="error.alert" transition="scale-transition">
      {{error.message}}
    </v-alert>
  </v-form>
</template>

<script>
export default {
  name: 'add-channel-form',
  props: ['success', 'error'],
  data() {
    return {
      valid: true,
      search: 'UC-lHJZR3Gqxm24_Vd_AJ5Yw', // Поле ввода
      searchRules: [v => !!v || 'Укажите ID канала'],
    };
  },
  methods: {
    submit() {
      if (this.$refs.form.validate()) {
        this.$emit('event', this.search);
        this.$refs.form.reset();
      }
    },
  },
};
</script>
