<template lang="pug">
video(ref="video" autoplay playsinline muted)
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";

@Component
export default class Camera extends Vue {
  async mounted() {
    try {
      const stream = await navigator.mediaDevices.getUserMedia({
        video: { facingMode: "user" },
        audio: false
      });
      const video = this.$refs.video as HTMLVideoElement;
      video.srcObject = stream;
    } catch (e) {
      window.alert(e);
    }
  }
  snapshot() {
    const video = this.$refs.video as HTMLVideoElement;
    const { videoWidth, videoHeight } = video;

    const canvas = document.createElement("canvas");
    canvas.width = videoWidth;
    canvas.height = videoHeight;

    const ctx = canvas.getContext("2d");
    if (!ctx) {
      throw new Error("画像の取得に失敗しました");
    }

    ctx.drawImage(video, 0, 0, videoWidth, videoHeight);

    return canvas;
  }
}
</script>
