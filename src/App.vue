<template>
  <div>
    <video id="video" ref="video" autoplay @loadeddata="loadVideo" />
    <canvas id="canvas" ref="canvas" />
  </div>
</template>

<script>
import "@mediapipe/face_detection";
import "@tensorflow/tfjs-core"; // Register WebGL backend.
import "@tensorflow/tfjs-backend-webgl";
import * as faceDetection from "@tensorflow-models/face-detection";

export default {
  name: "App",
  data() {
    return {
      constraints: {
        video: {
          width: 640,
          height: 480,
        },
        audio: false,
      },
      faceDetector: null,
    };
  },
  methods: {
    async setupCamera() {
      const stream = await navigator.mediaDevices.getUserMedia(
        this.constraints
      );
      this.$refs.video.srcObject = stream;
    },
    async setupModel() {
      const model = faceDetection.SupportedModels.MediaPipeFaceDetector;
      const detectorConfig = {
        runtime: "mediapipe",
        solutionPath: "https://cdn.jsdelivr.net/npm/@mediapipe/face_detection",
        // or 'base/node_modules/@mediapipe/face_detection' in npm.
      };
      this.faceDetector = await faceDetection.createDetector(
        model,
        detectorConfig
      );
    },
    async drawingFaceBound() {
      const video = this.$refs.video;
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext("2d");
      const faces = await this.faceDetector.estimateFaces(video);

      ctx.drawImage(video, 0, 0, video.offsetWidth, video.offsetHeight);
      faces.forEach((face) => {
        const { box } = face;

        ctx.strokeStyle = "red";
        ctx.lineWidth = 5;
        ctx.strokeRect(box.xMin, box.yMin, box.width, box.height);
      });

      window.requestAnimationFrame(this.drawingFaceBound);
    },
    loadVideo() {
      window.requestAnimationFrame(this.drawingFaceBound);
    },
    setupCanvas() {
      const canvas = this.$refs.canvas;
      canvas.width = this.constraints.video.width;
      canvas.height = this.constraints.video.height;
    },
  },
  async mounted() {
    await this.setupModel();
    await this.setupCamera();
    this.setupCanvas();
  },
};
</script>

<style>
#video {
  position: absolute;
  z-index: -1;
}
</style>
