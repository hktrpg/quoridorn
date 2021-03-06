<template>
  <div class="bgmCoreComponent">
    <div class="thumbnail">
      <img v-img="thumbnailData" draggable="false" alt="Not Found" :title="thumbnailTitle" @click="thumbnailClick">
    </div>
    <div class="bgmComponent">
      <div class="attrArea">
        <span class="tag" :title="'【タグ】\n' + tag">{{tag}}</span><!--
     --><span class="title" :title="'【タイトル】\n' + title">{{title}}</span><!--
     --><span class="icon loop" disabled v-if="isLoop"><i class="icon-infinite" title="ループ再生します"></i></span>
      </div>
      <div class="controlArea">
        <span class="icon play" :class="{isPlay: isPlay}" @click="changePlay()" v-show="isPlay"><i class="icon-play"></i></span>
        <span class="icon play" :class="{isPlay: isPlay}" @click="changePlay()" v-show="!isPlay"><i class="icon-pause"></i></span>
        <VolumeComponent
          :initVolume="initVolume"
          @mute="setMute"
          @volume="setVolume"
          ref="volumeComponent"/>
      </div>
      <div class="playLengthArea">
        <input class="playLength" :class="{isPlay: isPlay}" :style="playLengthStyle" type="range" min="0" :max="Math.round(bgmLength * 100) / 100" step="0.01" v-model="playLength" @input="seekTo">
        <span class="playLengthText">{{Math.round(playLength)}}/{{Math.round(bgmLength)}}</span>
      </div>
    </div>
  </div>
</template>

<script>
import { mapState, mapActions } from 'vuex'
import VolumeComponent from './VolumeComponent'

export default {
  name: 'bgmCoreComponent',
  props: {
    'tag': { type: String, required: true },
    'isLoop': { type: Boolean, required: true },
    'title': { type: String, required: true },
    'initVolume': { type: Number, required: true },
    'url': { type: String, required: true },
    'maxSecond': { type: Number, required: true }
  },
  components: {
    VolumeComponent: VolumeComponent
  },
  data () {
    return {
      jukeboxAudio: null,
      isYoutube: false,
      isPlay: true,
      playLength: 0,
      duration: 0,
      thumbnailData: ''
    }
  },
  mounted () {
    this.isYoutube = /www\.youtube\.com/.test(this.url)
    this.thumbnailData = `http://i.ytimg.com/vi/${window['getUrlParam']('v', this.url)}/default.jpg`
    this.$refs.volumeComponent.setVolume(this.initVolume)
    this.$refs.volumeComponent.setMute(false)
    this.$emit('mounted')
  },
  destroyed () {
    this.changePlay(false)
    this.$emit('destroyed')
  },
  methods: {
    ...mapActions([]),
    thumbnailClick () {
      window.open(this.url, '_blank')
    },
    audioMute () {
      this.$emit('mute', this.masterMute || this.$refs.volumeComponent.mute)
    },
    audioVolume () {
      this.$emit('volume', this.masterVolume * this.$refs.volumeComponent.volume)
    },
    setMute () { this.audioMute() },
    setVolume () { this.audioVolume() },
    changePlay (isPlay = !this.isPlay) {
      this.isPlay = isPlay
      this.$emit(isPlay ? 'play' : 'pause')
    },
    seekTo () {
      this.$emit('seekTo', this.playLength)
      this.changePlay(true)
    },
    timeUpdate (time) {
      this.playLength = time
      if (this.playLength >= this.bgmLength) {
        if (this.isLoop) {
          this.playLength = 0
          this.$emit('seekTo', 0)
        } else {
          this.$emit('pause')
          this.$emit('end')
        }
      }
    },
    pause () {
      this.isPlay = false
    },
    play () {
      this.isPlay = true
    },
    setDuration (duration) {
      this.duration = duration
    }
  },
  watch: {
    masterMute: {
      handler () { setTimeout(() => this.audioMute(), 0) },
      immediate: true
    },
    masterVolume: {
      handler () { setTimeout(() => this.audioVolume(), 0) },
      immediate: true
    }
  },
  computed: mapState({
    playLengthStyle () {
      const useColor = this.isPlay ? 'black' : '#8A084B'
      const per = this.playLength * 100 / this.bgmLength
      return {
        background: `linear-gradient(to right, ${useColor} 0%, ${useColor} ${per}%, rgba(100, 100, 100, 1) ${per}%, rgba(100, 100, 100, 1) 100%)`
      }
    },
    bgmLength () { return this.maxSecond > 0 ? this.maxSecond : this.duration },
    masterMute: state => state.private.display.jukeboxWindow.masterMute,
    masterVolume: state => state.private.display.jukeboxWindow.masterVolume,
    thumbnailTitle () {
      let title = `【タイトル】\n${this.title}`
      if (this.isYoutube) {
        title += `\n\n【URL】\n${this.url}`
      }
      return title
    }
  })
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.bgmCoreComponent {
  display: flex;
  flex-direction : row;
  min-height: 54px;
  margin-top: 4px;
  padding-top: 4px;
  border-top: 2px solid black;
}
.thumbnail {
  height: 54px;
  width: 72px;
  cursor: pointer;
}
.thumbnail > * {
  width: 100%;
  height: 100%;
}
.bgmComponent {
  flex: 1;
  display: flex;
  flex-direction : column;
  min-height: 54px;
}

.icon {
  color: black;
  font-size: 10px;
  padding: 0;
}
.icon i {
  box-sizing: border-box;
  border: 2px solid black;
  border-radius: 50%;
  width: 20px;
  height: 20px;
  display: inline-flex;
  justify-content: center;
  align-items: center;
}
.icon[disabled] i {
  background-color: lightgray;
}
.icon:not([disabled]) i:hover {
  border-color: #610B21;
  color: #610B21;
}
.icon:not([disabled]) i:active {
  border-color: #B40431;
  color: #B40431;
}
.icon.play i {
  border-radius: 50% 50% 0% 0%;
  cursor: pointer;
}
.icon.play:not(.isPlay) i {
  border-color: #8A084B;
  background-color: #8A084B;
  color: white;
}

.bgmComponent > div {
  flex-direction : row;
  position: relative;
}
.bgmComponent > div.attrArea {
  height: 1.5em;
}
.bgmComponent > div:not(.attrArea) {
  display: flex;
}

.bgmComponent > div > span {
  /* はみ出た文字省略 */
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.bgmComponent > div.attrArea > span.tag {
  display: inline-block;
  /* position: absolute; */
  font-size: 10px;
  padding: 0 2px;
  background-color: lightyellow;
  /* 罫線 */
  box-sizing: border-box;
  border: 1px solid black;
  border-radius: 5px;
  /* 縮小表示で限界を超えた小さいフォント対応 */
  max-width: calc((3em + 2px * 2) / 0.8);
  transform-origin: left;
  transform: scale(0.8);
  cursor: default;
}
.bgmComponent > div.attrArea > span.title {
  display: inline-block;
  position: absolute;
  left: calc(3em);
  text-align: left;
  font-weight: bold;
  cursor: default;
  width: calc(100% - 3em - 17px)
}
.bgmComponent > div.attrArea > .icon.loop {
  display: inline-block;
  position: absolute;
  top: 0;
  right: 0;
  margin: auto;
  transform-origin: right top;
  transform: scale(0.8);
}
.bgmComponent > div.attrArea > .icon.loop i {
  font-weight: bold;
}

.volumeText {
  position: absolute;
  right: 24px;
  bottom: 2px;
  transform-origin: right bottom;
  transform: scale(0.7);
  color: white;
  pointer-events: none;
}
.playLengthText {
  position: absolute;
  right: 24px;
  bottom: 6px;
  transform-origin: right bottom;
  transform: scale(0.7);
  color: white;
  pointer-events: none;
  user-select: none;
  -ms-user-select: none;
  -moz-user-select: none;
  -webkit-user-select: none;
}

input[type="range"] {
  -webkit-appearance:none;
  flex: 1;
  margin: 0;
  position: relative;
  background: rgba(0, 0, 0, 0);
  box-sizing: border-box;
  outline: 0;
  cursor: pointer;
}
input[type="range"]::-webkit-slider-thumb {
  position: relative;
  -webkit-appearance: none;
  cursor: pointer;
}
input[type="range"].volume::-webkit-slider-thumb {
  background: black;
  width: 6px;
  height: 20px;
  margin-top: -4px;
  margin-bottom: -4px;
  box-sizing: border-box;
  border: 2px solid black;
}

input[type="range"].playLength {
  height: 8px;
  position: relative;
  cursor: pointer;
  margin-bottom: 8px;
  flex: 1;
  width: 100%;
  border-color: black;
}
input[type=range].playLength::-webkit-slider-runnable-track{
  height: 8px;
  background: rgba(0,0,0,0);
  box-sizing: border-box;
}
input[type="range"].playLength::-webkit-slider-thumb {
  box-sizing: border-box;
  width: 6px;
  height: 16px;
  margin-top: 0px;
  background-color: black;
}
</style>
