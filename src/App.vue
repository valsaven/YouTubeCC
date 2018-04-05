<template>
<div id="app">
  <v-app>
    <v-container fliud fill-height grid-list-md text-xs-center>
      <v-layout row wrap align-center justify-center>
        <v-flex xs12>
          <h1>{{title}}</h1>
        </v-flex>
        <v-flex xs12>
          <v-form v-model="valid" ref="form" lazy-validation>
            <v-switch :label="`${isUsername ? 'По имени пользователя' : 'По ID канала'}`" color="indigo" v-model="isUsername"></v-switch>
            <v-text-field :label="`${isUsername ? 'Имя пользователя' : 'ID канала'}`" v-model="search" :rules="searchRules" required></v-text-field>
            <v-btn @click="submit" :disabled="!valid">Добавить</v-btn>
            <v-btn @click="clear">Очистить</v-btn>
          </v-form>
        </v-flex>
        <v-flex xs12>
          <v-list two-line subheader>
          <v-subheader>Текущий рейтинг</v-subheader>
          <v-select :items="orderValues" v-model="orderBy" label="Сортировать по ..." single-line></v-select>
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
import orderBy from 'lodash/orderBy';

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
      channels: [
      ],
      orderValues: [
        { text: 'Имени', value: 'name' },
        { text: 'Кол-ву подписчиков', value: 'subs' },
      ],
      isUsername: true,
      isMyChannelExists: false,
      myChannelName: null,
      orderBy: 'subs',
    };
  },
  computed: {
    selectedChannel() {
      return this.channels.find(channel => channel.name === this.myChannelName);
    },

    orderedChannels: function() {
      return orderBy(this.channels, this.orderBy, ['desc']);
    },
  },
  created() {
    const channels = JSON.parse(localStorage.getItem('channels'));

    if (channels) {
      this.channels = channels;
    }
  },
  methods: {
    selectChannel(channel) {
      if (this.isMyChannelExists) {
        this.isMyChannelExists = false;
        channel.isMy = false;
        this.myChannelName = '';
      } else {
        channel.isMy = true;
        this.myChannelName = channel.name;
        this.isMyChannelExists = true;
      }
    },
    removeChannel(channel) {
      this.channels = this.channels.filter(item => item.name !== channel.name);
    },
    toggleOrder(name) {
      this.currentOrder = name;
    },
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
