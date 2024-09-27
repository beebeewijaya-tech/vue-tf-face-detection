<template>
  <div>
    <video id="video" ref="video" autoplay @loadeddata="loadVideo"/>
    <canvas id="canvas" ref="canvas"/>
  </div>
</template>

<script setup>
import "@mediapipe/face_detection";
import "@tensorflow/tfjs-core"; // Register WebGL backend.
import "@tensorflow/tfjs-backend-webgl";
import * as faceDetection from "@tensorflow-models/face-detection";
import {onMounted, reactive, ref} from "vue";

const data = reactive({
  constraints: {
    video: {
      width: 640,
      height: 480,
    },
    audio: false,
  },
  faceDetector: null,
});
const video = ref(null);
const canvas = ref(null);
let faceDetector = null;

const setupCamera = async () => {
  video.value.srcObject = await navigator.mediaDevices.getUserMedia(
      data.constraints
  );
};

const setupModel = async () => {
  const model = faceDetection.SupportedModels.MediaPipeFaceDetector;
  const detectorConfig = {
    runtime: "mediapipe",
    solutionPath: "https://cdn.jsdelivr.net/npm/@mediapipe/face_detection",
    // or 'base/node_modules/@mediapipe/face_detection' in npm.
  };
  faceDetector = await faceDetection.createDetector(
      model,
      detectorConfig
  );
};

const drawingFaceBound = async () => {
  const innerVideo = video.value;
  const innerCanvas = canvas.value;
  const ctx = innerCanvas.getContext("2d");
  const faces = await faceDetector.estimateFaces(innerVideo);
  ctx.drawImage(innerVideo, 0, 0, innerVideo.offsetWidth, innerVideo.offsetHeight);
  faces.forEach((face) => {
    const {box} = face;

    ctx.strokeStyle = "red";
    ctx.lineWidth = 5;
    ctx.strokeRect(box.xMin, box.yMin, box.width, box.height);
  });
  window.requestAnimationFrame(drawingFaceBound);
};

const loadVideo = () => {
  window.requestAnimationFrame(drawingFaceBound);
};

const setupCanvas = () => {
  const innerCanvas = canvas.value;
  innerCanvas.width = data.constraints.video.width;
  innerCanvas.height = data.constraints.video.height;
};

onMounted(async () => {
  await setupModel();
  await setupCamera();
  setupCanvas();
});
</script>

<style>
#video {
  position: absolute;
  z-index: -1;
}
</style>
