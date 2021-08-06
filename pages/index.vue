<template>
  <div>
    音声<input type="file" required @change="selectedFile" /><br />
    <!-- 画像<input id="image" type="file" required /><br /> -->
    <v-slide-group v-model="model" class="pa-4" mandatory show-arrows>
      <v-slide-item
        v-for="imgsrc in imgs"
        :key="imgsrc"
        v-slot="{ active, toggle }"
      >
        <v-img
          contain
          :color="active ? 'primary' : 'grey lighten-1'"
          class="ma-4"
          height="150"
          width="200"
          :src="imgsrc"
          @click="toggle"
        >
          <v-row class="fill-height" align="center" justify="center">
            <v-scale-transition>
              <v-icon
                v-if="active"
                color="black"
                size="48"
                v-text="'mdi-close-circle-outline'"
              ></v-icon>
            </v-scale-transition>
          </v-row>
        </v-img>
      </v-slide-item>
    </v-slide-group>
    <v-btn @click="create">音声と画像から動画を作成する</v-btn>
    {{ conversionMethod }}
    <p>time:{{ currenttime }}</p>
    <video v-if="src" id="video" style="width: 70%" controls :src="src"></video>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
export type DataType = {
  src: String
  currenttime: Number
  model: any
  imgs: String[]
  uploadFile: any
  conversionMethod: string
}
export default Vue.extend({
  props: {},
  data(): DataType {
    return {
      src: '',
      currenttime: 0,
      model: null,
      conversionMethod: '', // mp4 or wabm
      uploadFile: null,
      imgs: [
        '/carousel/Frame158.png',
        '/carousel/averuni.png',
        '/carousel/Frame162.png',
      ],
    }
  },
  computed: {},
  created() {
    this.conversionMethod = this.isAvailableFFmpeg()
  },
  methods: {
    isAvailableFFmpeg() {
      // console.log('crossOriginIsolated:', crossOriginIsolated)
      // @ts-igonore
      // if (!crossOriginIsolated) {
      //   // Post SharedArrayBuffer
      //   return 'webm'
      // }
      if (!('SharedArrayBuffer' in window)) return 'webm'
      return 'mp4'
    },
    selectedFile(e: any) {
      // 選択された File の情報を保存しておく
      e.preventDefault()
      const files = e.target.files
      this.uploadFile = files[0]
    },
    create() {
      if (this.uploadFile == null) {
        alert('ファイルを選択してください')
        return
      }
      this.src = ''
      const tesimg = this.imgs[this.model]
      // await fetch('/img/ogp.png').then((response) => {
      //  return response.blob()
      // })
      const testaudio = this.uploadFile // '/sample_averuni/sample_averuni_basic.wav'

      console.log('tesimg:', tesimg)
      const self = this
      this.ffmpeg(
        '-y',
        '-loop',
        1,
        '-i',
        tesimg,
        '-i',
        testaudio,
        '-pix_fmt',
        'yuv420p',
        '-shortest',
        'out.mp4'
      ).then((mp4) => {
        self.src = URL.createObjectURL(mp4)
      })
      // .finally(() => (button.disabled = false))
    },
    async ffmpeg(...command: any[]) {
      //   const FFmpeg = require('@ffmpeg/ffmpeg')
      //   const ffmpeg = createFFmpeg({
      //     logger: (e) => console.log(e.message)
      //   })
      const { createFFmpeg, fetchFile } = require('@ffmpeg/ffmpeg')
      const ffmpeg = createFFmpeg({
        // corePath: '/ffmpeg-core',

        corePath: '/ffmpeg-core/ffmpeg-core.js',
        log: true,
        progress: (p: any) => (this.currenttime = p.time),
      })
      if (!ffmpeg.isLoaded()) {
        await ffmpeg.load()
      }
      // @ts-ignore
      for (var [i, v] of command.entries()) {
        if (v instanceof Blob) {
          ffmpeg.FS(
            'writeFile',
            String(i),
            new Uint8Array(await v.arrayBuffer())
          )
          v = i
        }
        if (i === 4) {
          //   console.log('v:', v)
          const a = await fetch(v).then((response) => {
            return response.arrayBuffer()
          })
          console.log('a:', a)
          await ffmpeg.FS('writeFile', String(i), new Uint8Array(a))
          v = i
        }
        command[i] = String(v)
      }

      await ffmpeg.run(...command)

      return new File([ffmpeg.FS('readFile', v).buffer], v)
    },
  },
})
</script>

<style lang="scss" scoped>
* {
  //
}
</style>
