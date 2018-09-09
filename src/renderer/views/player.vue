<template>
  <main>
    <video ref="video"></video>
    <audio ref="audio"></audio>
    <!-- <control-pane></control-pane> -->
  </main>
</template>

<script>
import { ipcRenderer } from "electron";
import player from "../player";
import ControlPane from "../components/player/ControlPane"
import util from "../../util"
import { parseSubtitle } from "../lib";
import { throws } from 'assert';
import { keymap } from '../config'

const SURPPORTED_VIDEO_FORMATE = ['mp4', 'ogg', 'mkv']
const SURPPORTED_SUBTITLE_FORMATE = ['srt', 'ass']
export default {
  data() {
    return {
      files: {
        video: {origin: null, url: ''},
        subtitle: {origin: null, text: ''}
      }
    }
  },
  components: { ControlPane },
  mounted() {
    player.bindVideoAudio(this.$refs.video, this.$refs.audio)
    this.bindDragEvent()
  },
  methods: {
    bindKeymap(keymap) {
      document.addEventListener('keyup', e => {
        keymap[e.keyCode]()
      })
    },
    bindDragEvent() {
      document.ondragover = document.ondrop = e => e.preventDefault()
      document.body.ondrop = e => this.handleDropOpenFile(e)
    },
    handleDropOpenFile(e) {
      e.preventDefault()
      let dragOpenFiles = e.dataTransfer.files
      if(dragOpenFiles.length > 2) {
        return
      }
      Array.from(dragOpenFiles).forEach(file => {
        if(this.checkVideoSupported(file)) {
          this.files.video = {
            origin: file,
            url: util.getUrlFromFile(file)
          }
          player.setVideoSource(util.getUrlFromFile(file))
        }
        if(this.checkSubtitleSupported(file)) {
          util.getTextFromFile(file).then(res => {
            console.log(res);
            this.files.subtitle = {
              origin: file,
              text: res
            }
            player.setSubtitle(parseSubtitle(util.getFileType(file.name), res)) 
          }).catch(res => {
            throw new Error(res)
          })
        }
     })
    },
    checkVideoSupported(file) {
      let fileType = util.getFileType(file.name)
      return SURPPORTED_VIDEO_FORMATE.includes(fileType)
    },
    checkSubtitleSupported(file) {
      let fileType = util.getFileType(file.name)
      return SURPPORTED_SUBTITLE_FORMATE.includes(fileType)
    }
  }
};
</script>

<style scoped>
main {
  height: 100vh;
}
video {
  width: 100%;
  height: 100%;
  background-color: black;
}
#waveform {
  position: absolute;
  bottom: 60px;
  width: 100%;
  height: 160px;
  background-color: white;
}
</style>