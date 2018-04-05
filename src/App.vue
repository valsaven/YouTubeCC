<template>
<div id="app">
  <v-app>
    <v-container fill-height grid-list-md text-xs-center>
      <v-layout column align-center justify-center>
        <v-flex xs12 md2>
          <h1>YouTubeCC</h1>
        </v-flex>
        <v-flex xs12 md4>
          <v-form v-model="valid" ref="form" lazy-validation>
            <v-switch :label="`${isUsername ? 'Добавить по имени пользователя' : 'Добавить по ID канала'}`" color="indigo" v-model="isUsername"></v-switch>
            <v-text-field :label="`${isUsername ? 'Имя пользователя' : 'ID канала'}`" v-model="search" :rules="searchRules" required></v-text-field>
            <v-btn @click="submit" :disabled="!valid">Добавить</v-btn>
            <v-btn @click="clear">Очистить</v-btn>
          </v-form>
        </v-flex>
        <v-flex xs12 md12>
          <v-list subheader>
            <v-subheader>Текущий рейтинг</v-subheader>
            <v-list-tile avatar v-for="channel in orderedChannels" :key="channel.name" :class="{ 'amber lighten-1': channel.isMy }">
              <v-list-tile-action v-if="!isMyChannelExists">
                <v-btn icon @click="selectChannel(channel)">
                  <v-icon>{{`${channel.isMy ? 'star' : 'person'}`}}</v-icon>
                </v-btn>
              </v-list-tile-action>
              <v-list-tile-action v-else-if="channel.isMy">
                <v-btn icon @click="selectChannel(channel)">
                  <v-icon>{{`${channel.isMy ? 'star' : 'person'}`}}</v-icon>
                </v-btn>
              </v-list-tile-action>
              <v-list-tile-content>
                <v-list-tile-title>{{channel.subs}}</v-list-tile-title>
                <v-list-tile-sub-title>{{channel.name}}</v-list-tile-sub-title>
              </v-list-tile-content>
              <v-list-tile-action v-if="!channel.isMy">
                <v-btn icon @click="removeChannel(channel)">
                  <v-icon color="red lighten-1">remove_circle</v-icon>
                </v-btn>
              </v-list-tile-action>
            </v-list-tile>
          </v-list>
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</div>
</template>

<script>
import axios from 'axios';

class User {
  constructor(name, subs, isMy = false) {
    this.name = name;
    this.subs = subs;
    this.isMy = isMy;
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
      isMyChannelExists: false,
      myChannelName: null,
      apiKey: 'AIzaSyAGC8ADxI5-M2yoi4Pt_AHE2FpaH2Y6xpM',
      fields: 'items/statistics/subscriberCount',
      timer: '',
    };
  },
  computed: {
    selectedChannel() {
      return this.channels.find(channel => channel.name === this.myChannelName);
    },
    orderedChannels() {
      return this.channels
        .sort((a, b) => {
          return Number(a.subs) - Number(b.subs);
        })
        .reverse();
    },
  },
  created() {
    const isMyChannelExists = JSON.parse(localStorage.getItem('isMyChannelExists'));
    const myChannelName = JSON.parse(localStorage.getItem('myChannelName'));
    const channels = JSON.parse(localStorage.getItem('channels'));

    if (isMyChannelExists) this.isMyChannelExists = isMyChannelExists;
    if (myChannelName) this.myChannelName = myChannelName;
    if (channels) {
      this.channels = channels;
      this.updateSubs(this.channels);
    }
  },
  methods: {
    saveChannels() {
      localStorage.setItem('channels', JSON.stringify(this.channels));
    },
    saveUserChannel() {
      localStorage.setItem('isMyChannelExists', JSON.stringify(this.isMyChannelExists));
      localStorage.setItem('myChannelName', JSON.stringify(this.myChannelName));
    },
    updateSubs(channels) {
      const updateChannel = async channel => {
        try {
          const res = await axios.get(
            `https://www.googleapis.com/youtube/v3/channels?part=statistics&forUsername=${
              channel.name
            }&fields=${this.fields}&key=${this.apiKey}`,
          );

          const data = res.data;
          let subs = null;

          if (data.items.length) {
            subs = data.items[0].statistics.subscriberCount;
          } else {
            throw new Error(this.isUsername ? 'Неверное имя пользователя.' : 'Неверный ID канала.');
          }

          channel.subs = Number(subs);
        } catch (e) {
          console.error(e);
        }
      };

      const updateChannels = channels => {
        for (let i = 0; i < channels.length; i++) {
          let channel = channels[i];
          updateChannel(channel);
        }

        return channels;
      };

      this.timer = setInterval(() => {
        this.channels = updateChannels(channels);
        this.saveChannels();
      }, 5000);
    },
    selectChannel(channel) {
      if (this.isMyChannelExists) {
        this.isMyChannelExists = false;
        channel.isMy = false;
        this.myChannelName = '';
        this.saveUserChannel();
      } else {
        this.isMyChannelExists = true;
        channel.isMy = true;
        this.myChannelName = channel.name;
        this.saveUserChannel();
      }

      this.saveChannels();
    },
    removeChannel(channel) {
      this.channels = this.channels.filter(item => item.name !== channel.name);
      this.saveChannels();
    },
    toggleOrder(name) {
      this.currentOrder = name;
    },
    getChannelData: async function(search) {
      try {
        const type = this.isUsername ? 'forUsername' : 'id';
        const res = await axios.get(
          `https://www.googleapis.com/youtube/v3/channels?part=statistics&${type}=${search}&fields=${
            this.fields
          }&key=${this.apiKey}`,
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
        this.saveChannels();
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
      clearInterval(this.timer);
    },
  },
};
</script>

<style scoped>
@import '../node_modules/vuetify/dist/vuetify.min.css';
</style>
