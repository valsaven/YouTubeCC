<template>
<div id="app">
  <v-app>
    <v-container fill-height grid-list-md text-xs-center>
      <v-layout column align-center justify-center>
        <!-- Header -->
        <v-flex xs2>
          <h1 class="header-title" @click="tile = !tile">YouTubeCC</h1>
        </v-flex>
        <!-- Форма добавления канала -->
        <v-flex xs4>
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
        </v-flex>
        <!-- Таблица рейтинга -->
        <v-flex xs12>
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
        </v-flex>
      </v-layout>
    </v-container>
  </v-app>
</div>
</template>

<script>
import axios from 'axios';
import addChannelForm from './components/addChannelForm';
import channelsList from './components/channelsList';

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
  name: 'app',
  components: { addChannelForm, channelsList },
  data() {
    return {
      title: 'YouTubeCC',
      valid: true,
      search: '', // Поле ввода
      searchRules: [v => !!v || 'Укажите ID канала'],
      myChannelID: '', // ID выбранного канала
      channelsIDs: [], // Массив с ID каналов
      channels: [], // Массив с объектами Channel
      tile: true, // Округление аватарок
      apiKey: 'AIzaSyCZtVMiNePq-6ag3d2MgJcSNVN_5b-t5e0',
      timer: '',
      success: {
        alert: false,
        message: '',
      },
      error: {
        alert: false,
        message: '',
      },
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
  methods: {
    saveChannels() {
      localStorage.setItem('channels', JSON.stringify(this.channels));
      localStorage.setItem('channelsIDs', JSON.stringify(this.channelsIDs));
    },
    saveUserChannel() {
      localStorage.setItem('myChannelID', JSON.stringify(this.myChannelID));
    },
    updateSubs() {
      const updateChannel = async channel => {
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
      };

      const updateChannels = channels => {
        for (let i = 0; i < channels.length; i++) {
          updateChannel(channels[i]);
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
    addChannel(id) {
      const getChannelInfo = async id => {
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
      };

      // Check for duplicates
      if (this.channelsIDs.includes(id)) {
        const errorMsg = 'Канал уже есть в списке.';

        this.error.message = errorMsg;
        this.error.alert = true;
        setTimeout(() => {
          this.error.alert = false;
        }, 3000);
      } else {
        getChannelInfo(id);
      }
    },
    removeChannel(channel) {
      this.channels = this.channels.filter(item => item.id !== channel.id);
      this.channelsIDs = this.channelsIDs.filter(item => item !== channel.id);
      this.saveChannels();
    },
    submit() {
      if (this.$refs.form.validate()) {
        this.addChannel(this.search);
        this.$refs.form.reset();
      }
    },
  },
};
</script>

<style scoped>
@import '../node_modules/vuetify/dist/vuetify.min.css';

.header-title {
  cursor: pointer;
}
</style>
