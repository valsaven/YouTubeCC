<template>
  <v-form v-model="valid" ref="form" lazy-validation>
    <v-text-field label="ID канала" v-model="search" :rules="searchRules" required></v-text-field>
    <v-btn @click="submit" :disabled="!valid">Добавить</v-btn>
    <alert-box v-bind="{success, error}"></alert-box>
  </v-form>
</template>

<script>
import alertBox from './alertBox';

export default {
  name: 'add-channel-form',
  components: { alertBox },
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
        this.$emit('submit', this.search);
        this.$refs.form.reset();
      }
    },
  },
};
</script>
