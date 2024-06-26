<template>
  <footer class="player-footer flex-row flex-middle flex-space">
    <!-- player controls -->
    <section class="player-controls flex-row flex-middle push-right"
             :class="{ 'disabled': !canPlay }">
      <button class="common-btn" @click="togglePlay()" aria-label="play">
        <i v-if="loading" class="fas fa-circle-notch fx fx-spin-right" key="wait"></i>
        <i v-else-if="playing" class="fa fa-stop fx fx-drop-in" key="stop"></i>
        <i v-else class="fa fa-play fx fx-drop-in" key="play"></i>
      </button>
      <div class="form-slider push-left">
        <i class="fa fa-volume-down" @click="volumeDown()"></i>
        <input aria-label="volume" class="common-slider" type="range" min="0.0" max="1.0" step="0.1" value="0.5"
               v-model="volume"/>
        <i class="fa fa-volume-up" @click="volumeUp()"></i>
      </div>
      <div class="text-clip push-left">
        <span>{{ timeDisplay }}</span>
      </div>
    </section>

    <!-- player links -->
    <section class="player-links text-nowrap">
      <animation-selection></animation-selection>
      <a v-if=config.twitter.username class="common-btn text-faded" rel="noreferrer"
         :href="'https://twitter.com/'+config.twitter.username" title="Twitter"
         target="_blank">
        <i class="fab fa-twitter"></i>
      </a> &nbsp;
      <a v-if=config.facebook.page_id class="common-btn text-faded" rel="noreferrer"
         :href="'https://fb.me/'+config.facebook.page_id"
         title="Facebook"
         target="_blank">
        <i class="fab fa-facebook"></i>
      </a>
    </section>
  </footer>
</template>

<script>
import animationSelection from '@/views/components/animationSelection.vue'
import _audio from '@/js/audio.js';
import emitter from 'tiny-emitter/instance'

export default {
  name: "footerPlayer",
  components: {
    animationSelection
  },
  props: {
    streamUrl: String,
    config: Object,
  },
  data: () => {
    return {
      playing: false,
      loading: false,
      volume: 0.5,
      errors: {},
      timeStart: 0,
      timeDisplay: '00:00:00',
      timeItv: null,

    }
  },
  watch: {
    volume() {
      _audio.setVolume(this.volume);
    },
    // watch playing status
    playing() {
      if (this.playing) {
        this.startClock();
      } else {
        this.stopClock();
      }
    },
  },
  methods: {
    // play audio stream for a channel
    playChannel() {
      console.log('playChannel');
      //if (this.playing ) return;
      this.loading = true;

      _audio.playSource(this.streamUrl);
      _audio.setVolume(this.volume);
    },


    // reset selected channel
    resetPlayer() {
      this.$parent.visible = false
      this.closeAudio();

    },
    canPlay() {
      return !!(this.streamUrl && !this.loading);
    },
    // toggle stream playback for current selected channel
    togglePlay() {
      //if (this.loading) return;
      if (this.playing) return this.closeAudio();
      return this.playChannel();
    },
    // setup audio routing and stream events
    setupAudio() {
      const a = _audio.setupAudio();

      a.addEventListener('waiting', e => {
        this.playing = false;
        this.loading = true;
      });
      a.addEventListener('playing', e => {
        this.playing = true;
        this.loading = false;
      });
      a.addEventListener('ended', e => {
        this.playing = false;
        this.loading = false;
      });
      a.addEventListener('error', e => {
        this.closeAudio();
        console.log(e);
        this.playing = false;
        this.loading = false;
      });
    },

    // close active audio
    closeAudio() {
      _audio.stopAudio();
      this.playing = false;
    },
    volumeUp() {
      if (this.volume + 0.1 > 1) return;
      this.volume += 0.1
    },
    volumeDown() {
      if (this.volume - 0.1 < 0) return;
      this.volume -= 0.1
    },
    // start tracking playback time
    startClock() {
      this.stopClock();
      this.timeStart = Date.now();
      this.timeItv = setInterval(this.updateClock, 1000);
      this.updateClock();
    },

    // update tracking playback time
    updateClock() {
      let p = n => (n < 10) ? '0' + n : '' + n;
      let elapsed = (Date.now() - this.timeStart) / 1000;
      let seconds = Math.floor(elapsed % 60);
      let minutes = Math.floor(elapsed / 60 % 60);
      let hours = Math.floor(elapsed / 3600);
      this.timeDisplay = p(hours) + ':' + p(minutes) + ':' + p(seconds);
    },

    // stop tracking playback time
    stopClock() {
      if (this.timeItv) clearInterval(this.timeItv);
      this.timeItv = null;
    },

    // clear timer refs
    clearTimers() {
      if (this.sto) clearTimeout(this.sto);
      if (this.itv) clearInterval(this.itv);
    },
    // on keyboard events
    onKeyboard(e) {
      const k = e.key || '';
      if (k === ' ' && this.stationId) return this.togglePlay();
    },

  },

  mounted() {
    this.setupAudio();
    emitter.on('resetPlayer', () => {
      this.resetPlayer();
    });
  },
  beforeDestroy() {
    this.closeAudio();
  },
  destroyed() {
    this.clearTimers();

  }
}
</script>

<style scoped>

</style>
