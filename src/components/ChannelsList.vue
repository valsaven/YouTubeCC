<template>
  <v-list
    class="v-subheader"
    :class="{ 'pulse': animated }"
    @animationend="animated = false"
  >
    <div class="v-subheader">Текущий рейтинг</div>
    <v-list-item
      v-for="(channel, index) in orderedChannels"
      :key="channel.id"
      avatar
      :class="{ 'amber lighten-1': channel.id === myChannelID }"
    >
      <div class="channel__index">
        {{ index }}
      </div>
      <!-- Аватарка канала -->
      <v-list-item-action class="channel__avatar">
        <v-btn
          icon
          @click="selectChannel(channel)"
        >
          <v-avatar
            :tile="tile"
            class="grey lighten-4"
          >
            <img
              :src="channel.thumbnailUrl"
              alt="avatar"
            >
          </v-avatar>
        </v-btn>
      </v-list-item-action>
      <!-- Основной блок информации о канале -->
      <v-list-item-content>
        <v-tooltip
          bottom
          max-width="400px"
        >
          <v-list-item-title slot="activator">
            {{ channel.subscriberCount }}
          </v-list-item-title>
          <v-list-item-subtitle slot="activator">
            {{ channel.title }}
          </v-list-item-subtitle>
          <span>{{ channel.description }}</span>
        </v-tooltip>
      </v-list-item-content>
      <!-- Кнопка удаления -->
      <v-list-item-action v-if="channel.id !== myChannelID">
        <v-btn
          icon="mdi-minus-circle"
          color="red lighten-1"
          @click="removeChannel(channel)"
        />
      </v-list-item-action>
    </v-list-item>
  </v-list>
</template>

<script lang="ts">
import axios from 'axios';
import { defineComponent } from 'vue';

class Channel {
  id: string;
  title: string;
  description: string;
  thumbnailUrl: string;
  subscriberCount: number;

  constructor(id = '', title = '', description = '', thumbnailUrl = '', subscriberCount = 0) {
    this.id = id;
    this.title = title;
    this.description = description;
    this.thumbnailUrl = thumbnailUrl;
    this.subscriberCount = subscriberCount;
  }
}

export default defineComponent({
  name: 'ChannelsList',

  props: ['tile', 'success', 'error'],

  data() {
    return {
      myChannelID: '', // ID выбранного канала
      channelsIDs: [] as string[], // Массив с ID каналов
      channels: [] as Channel[], // Массив с объектами Channel
      apiKey: 'AIzaSyAb3msPiGuwpjDXOCo3Hpau2vz3zKMkHfA',
      timer: 0,
      animated: false,
    };
  },

  computed: {
    isMyChannelExists(): boolean {
      return this.myChannelID.length !== 0;
    },

    orderedChannels(): any[] {
      const channels = [...this.channels];

      return channels
        .sort((a, b) => Number(a.subscriberCount) - Number(b.subscriberCount))
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

    async getChannelInfo(channelID: string) {
      try {
        const part = 'id,snippet,statistics';
        const res = await axios.get(
          `https://www.googleapis.com/youtube/v3/channels?part=${part}&id=${channelID}&fields=items(${part})&key=${
            this.apiKey
          }`,
        );
        const { data } = res;

        if (data.items.length) {
          const channel = data.items[0];
          const { id } = channel;
          const { title } = channel.snippet;
          const { description } = channel.snippet;
          const thumbnailUrl = channel.snippet.thumbnails.default.url;
          const subscriberCount = Number(channel.statistics.subscriberCount);

          this.channelsIDs.push(id);
          this.channels.push(new Channel(
            id,
            title,
            description,
            thumbnailUrl,
            subscriberCount,
          ));

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

    addChannel(id: string) {
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

    async updateChannel(channel: Channel) {
      try {
        const res = await axios.get(
          `https://www.googleapis.com/youtube/v3/channels?part=id,statistics&id=${
            channel.id
          }&fields=items(id,statistics/subscriberCount)&key=${this.apiKey}`,
        );

        const { data } = res;
        let subscriberCount = null;

        // Данные получены
        if (data.items.length) {
          const updatedChannel = data.items[0];
          ({ subscriberCount } = updatedChannel.statistics);
        } else {
          throw new Error('Ошибка при получении данных с сервера.');
        }

        channel.subscriberCount = subscriberCount;
      } catch (e) {
        console.error(e);
      }
    },

    updateSubs() {
      const updateChannels = (channels: Channel[]) => {
        for (let i = 0; i < channels.length; i += 1) {
          this.updateChannel(channels[i]);
        }

        return channels;
      };

      // TODO: В localstorage повесить время последнего запроса обновлений каналов. Если с того момента прошло 6 часов - обновить
      this.timer = setInterval(() => {
        this.channels = updateChannels(this.channels);
        this.saveChannels();
        this.animate();
      }, 300000);
    },

    selectChannel(channel: Channel) {
      // Можно было бы убрать, но хочется иметь возможность не выделять какой-либо канал вообще
      this.isMyChannelExists ? (this.myChannelID = '') : (this.myChannelID = channel.id);

      this.saveUserChannel();
      this.saveChannels();
    },

    removeChannel(channel: Channel) {
      this.channels = this.channels.filter(item => item.id !== channel.id);
      this.channelsIDs = this.channelsIDs.filter(item => item !== channel.id);
      this.saveChannels();
    },

    animate() {
      this.animated = true;
      setTimeout(() => {
        this.animated = false;
      }, 2000);
    },

    getAndSetDataFromLocalstorage(key: 'myChannelID' | 'channelsIDs' | 'channels') {
      const data = localStorage.getItem(key);

      if (data && JSON.parse(data)) {
        this[key] = JSON.parse(data);

        if (key === 'channels') {
          this.updateSubs();
        }
      }
    }
  },

  created() {
    this.getAndSetDataFromLocalstorage('myChannelID');
    this.getAndSetDataFromLocalstorage('channelsIDs');
    this.getAndSetDataFromLocalstorage('channels');
  },
});
</script>

<style scoped>
.channel__index {
  width: 24px;
}

.channel__avatar {
  height: 48px;
  margin: 0;
  width: 48px;
}

.pulse {
  box-shadow: 0 0 0 rgba(0, 10, 50, .4);
  animation: pulse 2s ease-in;
}

@keyframes pulse {
  0% {
    box-shadow: 0 0 0 0 rgba(0, 10, 50, .4);
  }

  70% {
    box-shadow: 0 0 0 10px rgba(0, 10, 50, 0);
  }

  100% {
    box-shadow: 0 0 0 0 rgba(0, 10, 50, 0);
  }
}
</style>
