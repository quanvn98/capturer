<template>
  <div
    class="capturer"
    ref="capturerRef"
    :style="{ height: `${cameraHeight}px` }"
  >
    <video v-show="mode === 'VIDEO'" ref="videoRef" @click="toggleImage">
      Video not available
    </video>
    <div
      class="overlay top"
      :style="{
        left: `${cameraPaddingX}px`,
        right: `${cameraPaddingX}px`,
        height: `${Math.min(cameraPaddingY, cameraHeight)}px`,
      }"
    ></div>
    <div class="overlay right" :style="{ width: `${cameraPaddingX}px` }"></div>
    <div
      class="overlay bottom"
      :style="{
        left: `${cameraPaddingX}px`,
        right: `${cameraPaddingX}px`,
        height: `${Math.min(cameraPaddingY, cameraHeight)}px`,
      }"
    ></div>
    <div class="overlay left" :style="{ width: `${cameraPaddingX}px` }"></div>
    <img
      v-show="mode === 'IMAGE'"
      ref="imageRef"
      alt="Image not available"
      :style="{
        left: `${cameraPaddingX}px`,
        right: `${cameraPaddingX}px`,
        top: `${cameraPaddingY}px`,
        bottom: `${cameraPaddingY}px`,
      }"
      @click="toggleImage"
    />
  </div>
</template>

<script>
export default {
  name: "MyCapturer",
  mounted() {
    this.$refs.videoRef.addEventListener("canplay", this.onCanPlay, false);

    window.navigator.mediaDevices
      .getUserMedia({ video: true, audio: false })
      .then((stream) => {
        this.$refs.videoRef.srcObject = stream;
        this.$refs.videoRef.play();
      })
      .catch((err) => {
        console.error(`An error occurred: ${err}`);
      });
  },
  destroyed() {
    this.$refs.videoRef.removeEventListener("canplay", this.onCanPlay, false);
  },
  data() {
    return {
      mode: "VIDEO",
      // tỉ lệ của vùng sáng (thẻ CCCD)
      windowRatio: 16 / 9,
      // khoảng padding 2 bên vùng sáng
      cameraPaddingX: 8,
      // chiều rộng vùng camera
      cameraWidth: 0,
      // chiều cao vùng camera
      cameraHeight: 0,
    };
  },
  computed: {
    cameraPaddingY() {
      return Math.round((this.cameraHeight - this.windowHeight) / 2);
    },
    windowWidth() {
      return this.cameraWidth - 2 * this.cameraPaddingX;
    },
    windowHeight() {
      return Math.round(this.windowWidth / this.windowRatio);
    },
  },
  methods: {
    onCanPlay() {
      this.cameraWidth = this.$refs.capturerRef.clientWidth;
      let videoRatio =
        this.$refs.videoRef.videoWidth / this.$refs.videoRef.videoHeight;
      this.cameraHeight = Math.round(this.cameraWidth / videoRatio);
    },
    toggleImage() {
      if (this.mode === "VIDEO") {
        let videoWidth = this.$refs.videoRef.videoWidth;
        let videoHeight = this.$refs.videoRef.videoHeight;
        let canvas = document.createElement("canvas");
        canvas.width = this.windowWidth;
        canvas.height = this.windowHeight;
        let context = canvas.getContext("2d");
        context.drawImage(
          this.$refs.videoRef,
          (this.cameraPaddingX * videoWidth) / this.cameraWidth,
          (this.cameraPaddingY * videoHeight) / this.cameraHeight,
          (this.windowWidth * videoWidth) / this.cameraWidth,
          (this.windowHeight * videoHeight) / this.cameraHeight,
          0,
          0,
          this.windowWidth,
          this.windowHeight
        );
        let src = canvas.toDataURL("image/png");
        this.$refs.imageRef.setAttribute("src", src);
        this.mode = "IMAGE";
      } else {
        this.mode = "VIDEO";
      }
    },
  },
};
</script>

<style scoped>
.capturer {
  position: relative;
}

.overlay {
  background-color: rgba(0, 0, 0, 0.5);
}
.overlay.top {
  position: absolute;
  top: 0;
}
.overlay.right {
  position: absolute;
  right: 0;
  height: 100%;
}
.overlay.bottom {
  position: absolute;
  bottom: 0;
}
.overlay.left {
  position: absolute;
  left: 0;
  height: 100%;
}

video {
  position: absolute;
  width: 100%;
  height: 100%;
}

img {
  position: absolute;
}
</style>
