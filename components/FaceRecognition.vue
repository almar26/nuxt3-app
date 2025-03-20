<script setup>
import { ref, onMounted, nextTick } from "vue";
import * as faceapi from "face-api.js";

const video1 = ref(null);
const video2 = ref(null);
const isLoaded = ref(false);

const loadModels = async () => {
  await faceapi.nets.tinyFaceDetector.loadFromUri("/models");
  await faceapi.nets.faceLandmark68Net.loadFromUri("/models");
  await faceapi.nets.faceRecognitionNet.loadFromUri("/models");
  await faceapi.nets.faceExpressionNet.loadFromUri("/models");
};

const startVideo = async (videoElement, deviceId) => {
  const stream = await navigator.mediaDevices.getUserMedia({
    video: { deviceId },
  });
  videoElement.srcObject = stream;
};

const getConnectedWebcams = async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  return devices.filter((device) => device.kind === "videoinput");
};

const setupCameras = async () => {
  const webcams = await getConnectedWebcams();
  if (webcams.length < 2) {
    alert("Two webcams are required!");
    return;
  }

  await startVideo(video1.value, webcams[0].deviceId);
  await startVideo(video2.value, webcams[1].deviceId);
};

const detectFaces = async () => {
  if (!isLoaded.value) return;
  const detections1 = await faceapi.detectAllFaces(video1.value, new faceapi.TinyFaceDetectorOptions());
  const detections2 = await faceapi.detectAllFaces(video2.value, new faceapi.TinyFaceDetectorOptions());

  console.log("Camera 1 Faces:", detections1);
  console.log("Camera 2 Faces:", detections2);
  
  requestAnimationFrame(detectFaces);
};

onMounted(async () => {
  await loadModels();
  await nextTick();
  await setupCameras();
  isLoaded.value = true;
  detectFaces();
});
</script>

<template>
  <div class="flex flex-col items-center gap-4">
    <h1 class="text-xl font-bold">Face Recognition with Two Webcams</h1>
    <div class="flex gap-4">
      <video ref="video1" autoplay class="w-1/2 border"></video>
      <video ref="video2" autoplay class="w-1/2 border"></video>
    </div>
  </div>
</template>

<style scoped>
video {
  width: 400px;
  height: 300px;
  border: 2px solid #000;
}
</style>