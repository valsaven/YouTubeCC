<template>
  <v-list subheader>
    <v-subheader>Текущий рейтинг</v-subheader>
    <v-list-tile avatar v-for="(channel, index) in orderedChannels" :key="channel.id" :class="{ 'amber lighten-1': channel.id === myChannelID }">
      <v-list-tile class="mr-4">{{index}}</v-list-tile>
      <!-- Аватарка канала -->
      <v-list-tile-action>
        <v-btn icon @click="selectChannel(channel)">
          <v-avatar :tile="tile" class="grey lighten-4">
            <img :src="channel.thumbnailUrl" alt="avatar">
          </v-avatar>
        </v-btn>
      </v-list-tile-action>
      <!-- Основной блок информации о канале -->
      <v-list-tile-content>
        <v-tooltip bottom max-width="400px">
          <v-list-tile-title slot="activator">{{channel.subscriberCount}}</v-list-tile-title>
          <v-list-tile-sub-title slot="activator">{{channel.title}}</v-list-tile-sub-title>
          <span>{{channel.description}}</span>
        </v-tooltip>
      </v-list-tile-content>
      <!-- Кнопка удаления -->
      <v-list-tile-action v-if="channel.id !== myChannelID">
        <v-btn icon @click="removeChannel(channel)">
          <v-icon color="red lighten-1">remove_circle</v-icon>
        </v-btn>
      </v-list-tile-action>
    </v-list-tile>
  </v-list>
</template>

<script>
import axios from 'axios';

class Channel {
  constructor(id = '', title = '', description = '', thumbnailUrl = '', subscriberCount = 0) {
    this.id = id;
    this.title = title;
    this.description = description;
    this.thumbnailUrl = thumbnailUrl;
    this.subscriberCount = subscriberCount;
  }
}

export default {
  name: 'channels-list',
  props: ['tile', 'success', 'error'],
  created() {
    const myChannelID = JSON.parse(localStorage.getItem('myChannelID'));
    const channelsIDs = JSON.parse(localStorage.getItem('channelsIDs'));
    const channels = JSON.parse(localStorage.getItem('channels'));

    if (myChannelID) this.myChannelID = myChannelID;
    if (channelsIDs) this.channelsIDs = channelsIDs;
    if (channels) {
      this.channels = channels;
      this.updateSubs();
    }
  },
  data() {
    return {
      myChannelID: '', // ID выбранного канала
      channelsIDs: [], // Массив с ID каналов
      channels: [], // Массив с объектами Channel
      apiKey: 'AIzaSyCZtVMiNePq-6ag3d2MgJcSNVN_5b-t5e0',
      timer: '',
    };
  },
  computed: {
    isMyChannelExists() {
      return this.myChannelID.length !== 0;
    },
    orderedChannels() {
      return this.channels
        .sort((a, b) => {
          return Number(a.subscriberCount) - Number(b.subscriberCount);
        })
        .reverse();
    },
  },
  methods: {
    saveChannels() {
      localStorage.setItem('channels', JSON.stringify(this.channels));
      localStorage.setItem('channelsIDs', JSON.stringify(this.channelsIDs));
    },
    saveUserChannel() {
      localStorage.setItem('myChannelID', JSON.stringify(this.myChannelID));
    },
    getChannelInfo: async function(id) {
      try {
        const part = 'id,snippet,statistics';
        const res = await axios.get(
          `https://www.googleapis.com/youtube/v3/channels?part=${part}&id=${id}&fields=items(${part})&key=${
            this.apiKey
          }`,
        );
        const data = res.data;

        if (data.items.length) {
          const channel = data.items[0];
          const id = channel.id;
          const title = channel.snippet.title;
          const description = channel.snippet.description;
          const thumbnailUrl = channel.snippet.thumbnails.default.url;
          const subscriberCount = Number(channel.statistics.subscriberCount);

          this.channelsIDs.push(id);
          this.channels.push(new Channel(id, title, description, thumbnailUrl, subscriberCount));

          // Сообщение об успехе
          this.success.message = 'Канал успешно добавлен.';
          this.success.alert = true;
          setTimeout(() => {
            this.success.alert = false;
          }, 3000);
          this.saveChannels();
        } else {
          // Сообщение об ошибке
          const errorMsg = 'Канала с таким ID не существует.';
          this.error.message = errorMsg;
          this.error.alert = true;
          setTimeout(() => {
            this.error.alert = false;
          }, 3000);
          throw new Error(errorMsg);
        }
      } catch (e) {
        console.error(e);
      }
    },
    addChannel(id) {
      // Check for duplicates
      if (this.channelsIDs.includes(id)) {
        const errorMsg = 'Канал уже есть в списке.';

        this.error.message = errorMsg;
        this.error.alert = true;
        setTimeout(() => {
          this.error.alert = false;
        }, 3000);
      } else {
        this.getChannelInfo(id);
      }
    },
    updateChannel: async function(channel) {
      try {
        let res = await axios.get(
          `https://www.googleapis.com/youtube/v3/channels?part=id,statistics&id=${
            channel.id
          }&fields=items(id,statistics/subscriberCount)&key=${this.apiKey}`,
        );

        let data = res.data;
        let subscriberCount = null;

        // Данные получены
        if (data.items.length) {
          let channel = data.items[0];
          subscriberCount = channel.statistics.subscriberCount;
        } else {
          throw new Error('Ошибка при получении данных с сервера.');
        }

        channel.subscriberCount = subscriberCount;
      } catch (e) {
        console.error(e);
      }
    },
    updateSubs() {
      const updateChannels = channels => {
        for (let i = 0; i < channels.length; i++) {
          this.updateChannel(channels[i]);
        }

        return channels;
      };

      this.timer = setInterval(() => {
        this.channels = updateChannels(this.channels);
        this.saveChannels();
      }, 10000);
    },
    selectChannel(channel) {
      // Можно было бы убрать, но хочется иметь возможность не выделять какой-либо канал вообще
      this.isMyChannelExists ? (this.myChannelID = '') : (this.myChannelID = channel.id);

      this.saveUserChannel();
      this.saveChannels();
    },
    removeChannel(channel) {
      this.channels = this.channels.filter(item => item.id !== channel.id);
      this.channelsIDs = this.channelsIDs.filter(item => item !== channel.id);
      this.saveChannels();
    },
  },
};
</script>
