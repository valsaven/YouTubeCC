<template>
    <v-app>
      <v-card
        class="d-flex flex-column fill-height grid-list-md text-xl-center"
      >
        <v-card
          class="d-flex flex-column align-center justify-center"
        >
          <!-- Header -->
          <div class="d-flex xs2">
            <h1
              class="header-title"
              @click="tile = !tile"
            >
              YouTubeCC
            </h1>
          </div>
          <!-- Форма добавления канала -->
          <div class="d-flex xs4">
            <add-channel-form
              v-bind="{ success, error }"
              @submit="addChannel"
            />
          </div>
          <!-- Таблица рейтинга -->
          <div class="d-flex xs12">
            <channels-list
              ref="chList"
              v-bind="{ tile, success, error }"
            />
          </div>
        </v-card>
      </v-card>
    </v-app>
</template>

<script lang="ts">
import AddChannelForm from './components/AddChannelForm.vue';
import ChannelsList from './components/ChannelsList.vue';

export default {
  name: 'App',
  components: { AddChannelForm, ChannelsList },
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
    addChannel(id: string) {
      //@ts-ignore
      this.$refs.chList.addChannel(id);
    },
  },
};
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
