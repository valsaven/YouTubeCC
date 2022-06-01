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

<script lang="ts">
import AlertBox from './AlertBox.vue';

export default {
  name: 'AddChannelForm',
  components: { AlertBox },
  props: ['success', 'error'],
  data() {
    return {
      valid: true,
      search: '', // Поле ввода
      searchRules: [(v: string) => !!v || 'Укажите ID канала'],
    };
  },
  methods: {
    submit() {
      //@ts-ignore
      if (this.$refs.form.validate()) {
        //@ts-ignore
        this.$emit('submit', this.search);
        //@ts-ignore
        this.$refs.form.reset();
      }
    },
  },
};
</script>
