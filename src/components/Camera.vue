<template lang="pug">
video(ref="video" autoplay muted :width="width" :height="height")
</template>

<script lang="ts">
import { Component, Prop, Vue } from "vue-property-decorator";

@Component
export default class Camera extends Vue {
  @Prop(Number) width = 640
  @Prop(Number) height = 640

  async init() {
    const stream = await navigator.mediaDevices.getUserMedia({
      video: { facingMode: "user", width: this.width, height: this.height },
      audio: false
    });
    const video = this.$refs.video as HTMLVideoElement;
    video.srcObject = stream;
  }
  snapshot() {
    const video = this.$refs.video as HTMLVideoElement;
    const { width, height } = video;

    const canvas = document.createElement("canvas");
    canvas.width = width;
    canvas.height = height;

    const ctx = canvas.getContext("2d");
    if (!ctx) {
      throw new Error('画像の取得に失敗しました')
    }

    ctx.drawImage(video, 0, 0, width, height);

    return canvas;
  }
}
</script>
