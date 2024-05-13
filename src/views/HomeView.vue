<template>
  <v-container>
    <h1 class="title">Rusu Radio Browser</h1>
    <div v-if="radios.length">
      <h2>Radio Stations</h2>
      <v-row>
        <v-col cols="12" sm="6" md="4" lg="3" v-for="radio in radios" :key="radio.stationuuid">
          <v-card class="mx-auto" max-width="400" hover dark elevation="10">
            <v-img :src="radio.favicon || require('@/assets/radiob.jpg')" aspect-ratio="1.75" class="white--text">
              <v-card-title class="fill-height align-end" v-text="radio.name"></v-card-title>
            </v-img>
            <v-card-actions>
              <v-btn color="orange" dark @click="togglePlayRadio(radio, index)">
                <svg-icon type="mdi" :path="currentPlayingUrl === radio.url ? mdiPause : mdiPlay"></svg-icon>
              </v-btn>
              <v-btn color="pink" dark @click="addToFavorites(radio)">
                <svg-icon type="mdi" :path="mdiHeart"></svg-icon>
              </v-btn>
              <span v-if="currentMediaInfoIndex === index">{{ currentMediaInfo }}</span>
              <span v-else>Podcast is playing</span>
            </v-card-actions>
          </v-card>
        </v-col>
      </v-row>
    </div>
    <div v-else>
      <p>Loading radio stations...</p>
    </div>
  </v-container>

  <!-- Player bar -->
  <v-bottom-nav v-if="currentPlayingRadio" class="player-bar">
  <div class="sound-wave-animation">
    <div class="sound-wave-bar"></div>
  </div>
  <v-btn color="orange" dark @click="togglePlayStop">
    {{ currentAudio && !currentAudio.paused ? "Stop" : "Play" }}
  </v-btn>
  <v-btn-text>{{ currentPlayingRadio.name }}</v-btn-text>
</v-bottom-nav>
</template>


<script>
import SvgIcon from '@jamescoyle/vue-icon';
import { mdiPlay, mdiPause, mdiHeart } from '@mdi/js';
import Hls from 'hls.js';


export default {
  name: 'HomeView',
  components: {
    SvgIcon
  },
  provide() {
    return {
      favorites: this.favorites,
    };
  },
  data() {
    return {
      radios: [],
      mdiPlay,
      mdiPause,
      mdiHeart,
      currentAudio: null,
      currentPlayingUrl: null,
      currentMediaInfo: null,
      currentPlayingRadio: null, // Make this reactive
      favorites: [],
    };
  },
  methods: {
    getRadios() {
      fetch('https://nl1.api.radio-browser.info/json/stations/search?limit=100&countrycode=IT&hidebroken=true&order=clickcount&reverse=true')
       .then(response => response.json())
       .then(data => {
          this.radios = data;
          console.log(data);
        });
    },
    togglePlayRadio(radio, index) {
  if (this.currentAudio && !this.currentAudio.paused) {
    this.currentAudio.pause();
    this.currentAudio = null;
    this.currentPlayingUrl = null;
    this.currentMediaInfoIndex = null;
  }

  const streamUrl = radio.hls === 1 ? radio.url : radio.url;
  if (streamUrl.includes('.m3u8')) {
    if (Hls.isSupported()) {
      const hls = new Hls();
      hls.loadSource(streamUrl);
      hls.attachMedia(this.$refs.audio);
      hls.on(Hls.Events.MEDIA_ATTACHED, () => {
        this.$refs.audio.play();
      });
      this.currentAudio = this.$refs.audio;
    } else {
      console.error('HLS is not supported in this browser.');
    }
  } else {
    this.currentAudio = new Audio(streamUrl);
    this.currentAudio.play();
  }
  this.currentPlayingUrl = streamUrl;
  this.currentPlayingRadio = radio; // Ensure this is updated
  this.currentMediaInfoIndex = index;
},
addToFavorites(radio) {
  radio.favorite = true; // Mark the radio as favorite
  const existsInFavorites = this.favorites.some(favorite => favorite.stationuuid === radio.stationuuid);
  if (!existsInFavorites) {
    this.favorites.push(radio); // Add the radio to favorites if it doesn't already exist
    localStorage.setItem('favorites', JSON.stringify(this.favorites));
  }
},
    togglePlayStop() {
  if (!this.currentAudio.paused) {
    this.currentAudio.pause();
  } else {
    this.currentAudio.play();
  }
}
  },
  created() {
    this.getRadios();
  },
  
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
  background-color: #444; /* Darker background */
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

.player-bar {
  position: fixed;
  bottom: 0;
  left: 50%; /* Center the bar */
  transform: translateX(-50%); /* Adjust for centering */
  width: 80%; /* Adjust width */
  height: 60px; /* Adjust height as needed */
  background-color: #333; /* Dark background */
  color: #fff; /* Text color */
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
  box-shadow: 0 -2px 5px rgba(0,0,0,0.3);
}

.player-bar .v-btn {
  margin-left: 10px;
}

.player-bar .v-icon {
  margin-left: 10px;
  font-weight: bold;
  text-align: right; /* Align text to the right */
}


.sound-wave-animation {
  width: 50px; /* Adjust width as needed */
  height: 40px; /* Adjust height as needed */
  position: absolute;
  left: 50%; /* Center horizontally */
  bottom: 50%; /* Align vertically at the bottom of the player bar */
  transform: translateX(-50%) translateY(50%);
  background: linear-gradient(to top, rgba(255, 255, 255, 0.6) 50%, rgba(255, 255, 255, 0) 100%); /* Gradient for sound wave */
  border-radius: 25px; /* Rounded shape */
  animation: soundWave 1s infinite alternate; /* Animation */
}

@keyframes soundWave {
  0% {
    height: 40px; /* Initial height */
  }
  100% {
    height: 60px; /* Maximum height */
  }
}

.v-dialog .v-card {
  padding: 20px;
  text-align: center;
}
.v-dialog img {
  max-width: 200px;
  margin: auto;
}
</style>
