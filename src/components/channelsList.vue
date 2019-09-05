<template>
  <v-list
    subheader
    :class="{ 'pulse': animated }"
    @animationend="animated = false"
  >
    <v-subheader>Текущий рейтинг</v-subheader>
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
          icon
          @click="removeChannel(channel)"
        >
          <v-icon color="red lighten-1">
            remove_circle
          </v-icon>
        </v-btn>
      </v-list-item-action>
    </v-list-item>
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
  name: 'ChannelsList',
  props: ['tile', 'success', 'error'],
  data() {
    return {
      myChannelID: '', // ID выбранного канала
      channelsIDs: [], // Массив с ID каналов
      channels: [], // Массив с объектами Channel
      apiKey: 'AIzaSyCZtVMiNePq-6ag3d2MgJcSNVN_5b-t5e0',
      timer: '',
      animated: false,
    };
  },
  computed: {
    isMyChannelExists() {
      return this.myChannelID.length !== 0;
    },
    orderedChannels() {
      const channels = [...this.channels];

      return channels
        .sort((a, b) => Number(a.subscriberCount) - Number(b.subscriberCount))
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
    async getChannelInfo(channelID) {
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
    async updateChannel(channel) {
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
      const updateChannels = (channels) => {
        for (let i = 0; i < channels.length; i += 1) {
          this.updateChannel(channels[i]);
        }

        return channels;
      };

      this.timer = setInterval(() => {
        this.channels = updateChannels(this.channels);
        this.saveChannels();
        this.animate();
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
    animate() {
      this.animated = true;
      setTimeout(() => {
        this.animated = false;
      }, 2000);
    },
  },
};
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
