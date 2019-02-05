<template>
  <v-form
    ref="form"
    v-model="valid"
    lazy-validation
  >
    <v-text-field
      v-model="search"
      label="ID канала"
      :rules="searchRules"
      required
    />
    <v-btn
      :disabled="!valid"
      @click="submit"
    >
      Добавить
    </v-btn>
    <alert-box v-bind="{success, error}" />
  </v-form>
</template>

<script>
import alertBox from './alertBox.vue';

export default {
  name: 'AddChannelForm',
  components: { alertBox },
  props: ['success', 'error'],
  data() {
    return {
      valid: true,
      search: '', // Поле ввода
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
