<template>
  <div>
    <!-- <video v-if="src" id="video" style="width: 70%" controls :src="src"></video> -->
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
export type DataType = {
      audio:Audio|null
      canvas:any
}
export default Vue.extend({
  props: {},
  data(): DataType {
    return {
      audio:null,
canvas:null
    }
  },
  computed: {},
  created() {},
  methods: {
    create() {
      // Videoストリーム
      // layer.getCanvas()._canvas は konva.js のcanvas参照
      const canvasStream = //layer.getCanvas()._canvas.captureStream(FPS)

      // Audioストリーム
      // オーディオライブラリにはhowler.jsを使用
      const audioContext = Howler.ctx
      const audioNode = Howler.masterGain
      const destination = audioContext.createMediaStreamDestination()
      audioNode.connect(destination)
      const audioStream = destination.stream

      // それぞれのストリームを結合したMediaStreamを作成
      const mediaStream = new MediaStream()
      ;[canvasStream, audioStream].forEach((stream) => {
        stream.getTracks().forEach((track) => mediaStream.addTrack(track))
      })

      // webmのフォーマットでMediaRecorderを作成
      const recorder = new MediaRecorder(mediaStream, {
        mimeType: 'video/webm;codecs=vp9',
      })

      // データが利用可能になった際に動画ファイルのダウンロードリンクを表示
      recorder.ondataavailable = function (e) {
        const videoBlob = new Blob([e.data], { type: e.data.type })
        const dataUrl = window.URL.createObjectURL(videoBlob)
        window.webmData = dataUrl
        const anchor = document.querySelector('#downloadlink')
        anchor.download = `movie${Date.now()}.webm`
        anchor.href = dataUrl
        anchor.style.display = 'block'
      }
    },
  },
})
</script>

<style lang="scss" scoped>
* {
  //
}
</style>
