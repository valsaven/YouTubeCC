<template>
  <div id="app">
    <v-app>
      <v-container
        fill-height
        grid-list-md
        text-xs-center
      >
        <v-layout
          column
          align-center
          justify-center
        >
          <!-- Header -->
          <v-flex xs2>
            <h1
              class="header-title"
              @click="tile = !tile"
            >
              YouTubeCC
            </h1>
          </v-flex>
          <!-- Форма добавления канала -->
          <v-flex xs4>
            <add-channel-form
              v-bind="{success, error}"
              @submit="addChannel"
            />
          </v-flex>
          <!-- Таблица рейтинга -->
          <v-flex xs12>
            <channels-list
              ref="chList"
              v-bind="{tile, success, error}"
            />
          </v-flex>
        </v-layout>
      </v-container>
    </v-app>
  </div>
</template>

<script>
import addChannelForm from './components/addChannelForm.vue';
import channelsList from './components/channelsList.vue';

export default {
  name: 'App',
  components: { addChannelForm, channelsList },
  data() {
    return {
      title: 'YouTubeCC',
      tile: true, // Округление аватарок
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
  methods: {
    addChannel(id) {
      this.$refs.chList.addChannel(id);
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
