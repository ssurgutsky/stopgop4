<template>
  <div>
    <audio autoplay :loop="loop" ref="audioPlayer" @ended="onAudioEnded">
      <source type="audio/mp3" />
    </audio>
    <audio autoplay ref="ambientPlayer">
      <source type="audio/mp3" />
    </audio>
    <audio autoplay ref="musicPlayer" @ended="onMusicEnded">
      <source type="audio/mp3" />
    </audio>
    <audio autoplay ref="bgndMusicPlayer" @ended="onBgndMusicEnded">
      <source type="audio/mp3" />
    </audio>
    <audio autoplay ref="sfxPlayer">
      <source type="audio/mp3" />
    </audio>
  </div>
</template>

<script>
import CacheController from '@/components/controller/CacheController.js'
import { setTimeout } from 'timers'

export default {
  name: 'AudioPlayer',
  data () {
    return {
      audioPlayer: null,
      ambientPlayer: null,
      musicPlayer: null,
      bgndMusicPlayer: null,
      sfxPlayer: null,
      currentAudioName: '',
      currentAmbientName: '',
      currentMusicName: '',
      silentTimerId: 0,
      loop: false
    }
  },
  mounted () {
    if (!this.audioPlayer) {
      this.initialize()
    }
  },
  methods: {
    initialize () {
      this.audioPlayer = this.$refs.audioPlayer
      this.ambientPlayer = this.$refs.ambientPlayer
      this.musicPlayer = this.$refs.musicPlayer
      this.bgndMusicPlayer = this.$refs.bgndMusicPlayer
      this.sfxPlayer = this.$refs.sfxPlayer
    },

    playAudio (name, loop) {
      console.log(name)

      this.clearSilentTimer()

      this.currentAudioName = name

      if (name.toUpperCase().includes('SILENT_')) {
        let seconds = name.toUpperCase().replace('SILENT_', '')
        this.silentTimerId = setTimeout(this.onAudioEnded, seconds * 1000)
        return
      }
      if (this.musicPlayer.src !== '') {
        this.musicPlayer.volume = 0.05
      }
      if (this.bgndMusicPlayer.src !== '') {
        this.bgndMusicPlayer.volume = 0.05
      }
      this.audioPlayer.loop = loop
      this.audioPlayer.src = this.getAudioSrc(name)
      this.audioPlayer.pause()
      setTimeout(() => {
        this.audioPlayer.play()
      }, 10)
    },

    getAudioSrc (name) {
      if (name.indexOf('http') >= 0) {
        return name
      }
      let asset = CacheController.getAssetByName(CacheController.CATEGORY_AUDIO, name)
      if (asset) {
        return asset
      }
      return this.getAudioPathByName(name)
    },

    getAudioPathByName (name) {
      let result = name
      if (result.indexOf('.mp3') < 0) {
        result = result + '.mp3'
      }
      return require('@/assets/audio/' + result)
    },

    stopAudio () {
      this.audioPlayer.pause()
      if (this.musicPlayer.src !== '') {
        this.musicPlayer.volume = 0.4
      }
      if (this.bgndMusicPlayer.src !== '') {
        this.bgndMusicPlayer.volume = 0.4
      }
    },

    clearSilentTimer () {
      clearTimeout(this.silentTimerId)
      this.silentTimerId = 0
    },

    onAudioEnded () {
      //      console.log('audioEnded', this.currentAudioName)
      this.$emit('audioEnded', this.currentAudioName)
    },

    playAmbient (name) {
      console.log('ambient', name)

      if (name === '' || name === this.currentAmbientName) {
        return
      }

      this.currentAmbientName = name

      if (name.toUpperCase() === 'NONE' || name.toUpperCase() === 'OFF') {
        this.ambientPlayer.src = ''
        return
      }

      this.ambientPlayer.loop = true
      this.ambientPlayer.src = this.getAudioSrc(name)
      this.ambientPlayer.pause()
      setTimeout(() => {
        this.ambientPlayer.play()
      }, 10)
    },

    playMusic (name) {
      console.log('playMusic', name)

      if (name === '') {
        return
      }

      this.currentMusicName = name

      if (name.toUpperCase() === 'NONE' || name.toUpperCase() === 'OFF') {
        this.musicPlayer.src = ''
        return
      }

      this.musicPlayer.src = this.getAudioSrc(name)
      this.musicPlayer.pause()
      setTimeout(() => {
        this.musicPlayer.play()
      }, 10)
    },

    onMusicEnded () {
      //      console.log('musicEnded', this.currentMusicName)
      this.$emit('musicEnded', this.currentMusicName)
    },

    playBgndMusic (name) {
      console.log('playBgndMusic', name)

      if (name === '') {
        return
      }

      this.currentBgndMusicName = name

      if (name.toUpperCase() === 'NONE' || name.toUpperCase() === 'OFF') {
        this.bgndMusicPlayer.src = ''
        return
      }

      this.bgndMusicPlayer.src = this.getAudioSrc(name)
      this.bgndMusicPlayer.pause()
      setTimeout(() => {
        this.bgndMusicPlayer.play()
      }, 10)
    },

    onBgndMusicEnded () {
      //      console.log('musicEnded', this.currentMusicName)
      this.$emit('bgndMusicEnded', this.currentBgndMusicName)
    },

    playSFX (name) {
      console.log('SFX', name)
      this.sfxPlayer.src = this.getAudioSrc(name)
      this.sfxPlayer.pause()
      setTimeout(() => {
        this.sfxPlayer.play()
      }, 10)
    },

    stopSFX () {
      this.sfxPlayer.pause()
    }
  }
}
</script>

<style scoped>
</style>
