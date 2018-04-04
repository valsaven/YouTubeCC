<template>
  <div id="app">
    <v-app>
      <h1>{{title}}</h1>
      <v-form v-model="valid" ref="form" lazy-validation>
        <v-switch
          :label="`${isUsername ? 'По имени пользователя' : 'По ID канала'}`"
          color="indigo"
          v-model="isUsername"
        ></v-switch>
        <v-text-field
          :label="`${isUsername ? 'Имя пользователя' : 'ID канала'}`"
          v-model="search"
          :rules="searchRules"
          required
        ></v-text-field>
        <v-btn @click="submit" :disabled="!valid">Добавить</v-btn>
        <v-btn @click="clear">Очистить</v-btn>
      </v-form>
    </v-app>
  </div>
</template>

<script>
import axios from 'axios';

class User {
  constructor(name, subs, isMe = false) {
    this.name = name;
    this.subs = subs;
    this.isMe = isMe;
  }
}

export default {
  name: 'app',
  data() {
    return {
      title: 'YouTubeCC',
      valid: true,
      search: '',
      searchRules: [
        v => !!v || (this.isUsername ? 'Укажите имя пользователя' : 'Укажите ID канала'),
      ],
      channels: [],
      isUsername: true,
    };
  },
  created() {},
  methods: {
    getChannelData: async function(search) {
      try {
        const apiKey = 'AIzaSyAGC8ADxI5-M2yoi4Pt_AHE2FpaH2Y6xpM';
        const fields = 'items/statistics/subscriberCount';
        const type = this.isUsername ? 'forUsername' : 'id';

        const res = await axios.get(
          `https://www.googleapis.com/youtube/v3/channels?part=statistics&${type}=${search}&fields=${fields}&key=${apiKey}`,
        );

        const data = res.data;
        let subs = null;

        if (data.items.length) {
          subs = data.items[0].statistics.subscriberCount;
        } else {
          throw new Error(this.isUsername ? 'Неверное имя пользователя.' : 'Неверный ID канала.');
        }

        this.search = '';
        this.channels.push(new User(search, subs));
      } catch (e) {
        console.error(e);
      }
    },
    submit() {
      if (this.$refs.form.validate()) {
        this.getChannelData(this.search);
      }
    },
    clear() {
      this.$refs.form.reset();
    },
  },
};
</script>

<style scoped>
@import '../node_modules/vuetify/dist/vuetify.min.css';
</style>
