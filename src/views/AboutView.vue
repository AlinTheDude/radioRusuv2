<template>
  <div class="about">
    <h1 class="title">Favorites</h1>
    <div v-if="favorites && favorites.length">
      <h2>Favorite Radio Stations</h2>
      <v-row>
        <v-col cols="12" sm="6" md="4" lg="3" v-for="radio in favorites" :key="radio.stationuuid">
          <v-card class="mx-auto" max-width="400" hover dark elevation="10">
            <v-img :src="radio.favicon || require('@/assets/radioImg.jpg')" aspect-ratio="1.75" class="white--text">
              <v-card-title class="fill-height align-end" v-text="radio.name"></v-card-title>
            </v-img>
            <v-card-actions>
              <v-btn color="orange" dark @click="togglePlayRadio(radio)">
                <svg-icon type="mdi" :path="currentPlayingUrl === radio.url? mdiPause : mdiPlay"></svg-icon>
              </v-btn>
              <v-btn color="pink" dark @click="removeFromFavorites(radio)">
                <svg-icon type="mdi" :path="mdiHeartOutline"></svg-icon>
              </v-btn>
              <span v-if="currentMediaInfo">{{ currentMediaInfo }}</span>
              <span v-else>Podcast is playing</span>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </div>
    <div v-else>
      <p>No favorites yet.</p>
    </div>
  </div>
</template>

<script>
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiPlay, mdiPause, mdiHeart, mdiHeartOutline } from '@mdi/js';

export default {
  name: 'AboutView',
  components: {
    SvgIcon
  },
  inject: ['favorites'],
  data() {
    return {
      mdiPlay,
      mdiPause,
      mdiHeart,
      mdiHeartOutline,
      currentAudio: null,
      currentPlayingUrl: null,
      currentMediaInfo: null,
    };
  },
  methods: {
    togglePlayRadio(radio) {
      console.log('Toggling play for radio:', radio.name); // Debugging
      if (this.currentPlayingUrl === radio.url) {
        if (this.currentAudio) {
          this.currentAudio.pause();
        }
        this.currentAudio = null;
        this.currentPlayingUrl = null;
        this.currentMediaInfo = null;
      } else {
        const streamUrl = radio.hls === 1 ? radio.url : radio.url;
        this.currentAudio = new Audio(streamUrl);
        this.currentAudio.play();
        this.currentPlayingUrl = radio.url;
        this.currentMediaInfo = radio.name;
        console.log('Playing radio:', radio.name);
      }
    },
    retrieveFavorites() {
  const favorites = JSON.parse(localStorage.getItem('favorites')) || [];
  this.favorites = favorites;
},
removeFromFavorites(radio) {
  console.log('Removing from favorites:', radio.name); // Debugging
  const index = this.favorites.findIndex(r => r.stationuuid === radio.stationuuid);
  if (index!== -1) {
    this.favorites.splice(index, 1);
    // Update localStorage to reflect the change
    localStorage.setItem('favorites', JSON.stringify(this.favorites));
  }
},

  },
  created() {
    this.retrieveFavorites();
  }
};
</script>

<style scoped>
.title {
  font-family: '70\'s Disco Personal Use', cursive;
  color: #fff;
  text-align: center;
  margin-bottom: 20px;
}

.v-card {
  margin-bottom: 20px;
  transition: transform 0.3s ease-in-out;
  background-color: #444;
  box-shadow: 0 4px 8px 0 rgba(0,0,0,0.2), 0 6px 20px 0 rgba(0,0,0,0.19);
}

.v-card:hover {
  transform: scale(1.05);
}

.v-card-title {
  font-size: 1.2rem;
  font-weight: bold;
  color: #fff;
}

.v-card-actions {
  justify-content: center;
}

.v-btn {
  margin-top: 10px;
}
</style>