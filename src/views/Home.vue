<template lang="pug">
#main(ref="main")
  camera.camera(ref="camera" :width="videoWidth" :height="videoHeight")
  .loading(v-if="scene === 0") Loading...
  .scene(v-if="scene === 1")
    h1.title 変顔バトル
    .msg ふつう顔を見せてください
    .buttons
      a.button(@click="initFace") ふつう顔を撮る
  .scene(v-if="scene === 2")
    img.latest-face(v-if="initialFace.url" :src="initialFace.url")
    .msg 5 回連続で変顔を作ってください
    .buttons
      a.button(@click="start") Let's 変顔!
      a.button(@click="scene = 1") ふつう顔を撮り直す
  .scene(v-if="[3, 4].includes(scene)")
    template(v-if="scene === 3")
      .msg {{ faces.length + 1 }} 枚目...
    template(v-if="scene === 4")
      .result
        .header-msg あなたの変顔スコアは...
        .score(:style="scoreStyle") {{ animatedScore }}
        .footer-msg {{ scoreMessage }}
      .buttons
        a.button(@click="replay") もういちど
    .faces
      img.face(v-for="face in faces" :src="face.url")
    
</template>

<script lang="ts">
import { TweenLite } from "gsap";
import * as faceapi from "face-api.js";
import { Component, Vue } from "vue-property-decorator";
import Camera from "@/components/Camera.vue";

const FACE_API_MODEL_URL = "/face-api-models";

interface FaceData {
  canvas: HTMLCanvasElement
  url: string
}

@Component({
  components: {
    Camera
  }
})
export default class Home extends Vue {
  scene = 0
  initialFace: FaceData = {} as FaceData
  faces: FaceData[] = []
  videoWidth: number = 0
  videoHeight: number = 0
  timer: number = 0
  scoresInitFace: number[] = []
  scoresOriginality: number[] = []
  totalScore = 0

  mounted() {
    this.initModels()
    this.onResize()
    window.addEventListener('resize', this.onResize);
    this.$nextTick(() => {
      this.camera.init()
    })
  }

  get camera(): Camera {
    return this.$refs.camera as Camera
  }

  get animatedScore() {
    return Math.floor(this.totalScore)
  }

  get scoreStyle() {
    const fs = Math.floor(this.animatedScore / 100) * 10 + 50;
    const r = 255 - Math.floor(255 / (this.animatedScore + 1));
    return {
      fontSize: `${fs}px`,
      color: `rgb(${r},0,0)`
    };
  }

  get scoreMessage() {
    if (this.animatedScore < 400) {
      return "平凡"
    } else if (this.animatedScore < 800) {
      return "全米が笑った";
    } else {
      return "変顔神";
    }
  }

  onResize() {
    const main = this.$refs.main as HTMLElement
    this.videoWidth = main.clientWidth
    this.videoHeight = main.clientHeight
  }

  async initModels() {
    try {
      await faceapi.loadSsdMobilenetv1Model(FACE_API_MODEL_URL);
      await faceapi.loadFaceRecognitionModel(FACE_API_MODEL_URL);
    } finally {
      this.scene = 1
    }
  }

  async initFace() {
    const canvas = await this.getFace()
    this.initialFace = {
      canvas,
      url: canvas.toDataURL()
    }
    this.scene = 2
  }

  start() {
    this.scene = 3
    this.timer = setInterval(async () => {
      const canvas = await this.getFace()
      this.faces.push({
        canvas,
        url: canvas.toDataURL()
      })
      if (this.faces.length === 5) {
        clearInterval(this.timer)
        this.scene = 4
        this.calcScore()
      }
    }, 1500)
  }

  async calcScore() {
    const desc = await faceapi.computeFaceDescriptor(this.initialFace.canvas) as Float32Array
    const descs = []
    let totalScore = 0
    for (const face of this.faces) {
      const desc2 = await faceapi.computeFaceDescriptor(face.canvas) as Float32Array
      const distance = faceapi.euclideanDistance(desc, desc2)
      const score = Math.floor((1 - distance) * 100)
      this.scoresInitFace.push(score)
      descs.push(desc2)
      totalScore += score
    }
    for (let i = 0; i < descs.length - 1; i++) {
      const a = descs[i]
      for (let j = i + 1; j < descs.length; j++) {
        const b = descs[j]
        const distance = faceapi.euclideanDistance(a, b)
        const score = Math.floor((1 - distance) * 100)
        this.scoresOriginality.push(score)
        totalScore += score
      }
    }

    TweenLite.to(this.$data, 3, { totalScore });
  }
  
  async getFace() {
    const input = this.camera.snapshot()
    const detection = await faceapi.detectSingleFace(input)
    if (!detection) {
      throw new Error('顔取得に失敗しました')
    }
    const { box } = detection
    const [extractedCanvas] = await faceapi.extractFaces(input, [box])
    return extractedCanvas
  }

  replay() {
    this.initialFace = {} as FaceData
    this.faces = []
    this.timer = 0
    this.scoresInitFace = []
    this.scoresOriginality = []
    this.totalScore = 0
    this.scene = 1
  }
}
</script>

<style lang="sass" scoped>
#main
  position: relative
  margin: auto
  width: 100vw
  max-width: 640px
  height: 100vh
  max-height: 720px
  color: black

.latest-face
  display: block
  position: absolute
  right: 0
  top: 0
  width: 80px
  height: 80px
  object-fit: cover

.camera,
.loading,
.scene
  position: absolute
  top: 0
  bottom: 0
  left: 0
  right: 0

.loading
  background-color: rgba(255,255,255,.7)
  display: flex
  justify-content: center
  align-items: center
  font-size: 3rem

.scene
  display: grid
  grid-template-rows: 100px 1fr 60px 60px 80px
  .title
    background-color: rgba(255,255,255,.7)
    text-align: center
    margin: auto 0
    padding: 10px 0
    grid-row: 1
  .result
    grid-row: 2
    position: relative
    display: flex
    align-items: center
    justify-content: center
    text-align: center
    background-color: rgba(255,255,255,.7)
    margin: 20px
    > .header-msg
      position: absolute
      width: 100%
      top: 20px
    > .footer-msg
      position: absolute
      width: 100%
      bottom: 20px
      font-size: 2.5rem
      font-weight: bold
  .msg
    background-color: rgba(255,255,255,.7)
    text-align: center
    grid-row: 3
    margin: auto
    padding: 10px
  .buttons
    grid-row: 4
    display: flex
    align-items: center
    justify-content: center
  .faces
    grid-row: 5
    display: grid
    grid-template-columns: repeat(5, 1fr)
    .face
      width: 100%
      height: 100%
      object-fit: cover
      order: 0
</style>
