<template lang="pug">
video(ref="video" autoplay playsinline muted)
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Camera extends Vue {
  width = 1280
  height = 720

  async mounted() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: "user" },
        audio: false
      });
      const video = this.$refs.video as HTMLVideoElement;
      video.srcObject = stream;
      const s = stream.getVideoTracks()[0].getSettings()
      const { width, height } = s
      this.width = width || 1280
      this.height = height || 720
    } catch (e) {
      window.alert(e)
    }
  }
  snapshot() {
    const video = this.$refs.video as HTMLVideoElement;

    const canvas = document.createElement("canvas");
    canvas.width = this.width;
    canvas.height = this.height;

    const ctx = canvas.getContext("2d");
    if (!ctx) {
      throw new Error('画像の取得に失敗しました')
    }

    ctx.drawImage(video, 0, 0, this.width, this.height);

    return canvas;
  }
}
</script>
