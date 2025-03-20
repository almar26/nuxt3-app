<script setup>
import * as faceapi from 'face-api.js'

const video = ref(null)
const video2 = ref(null)
const isLoading = ref(true)

onMounted(async () => {
  await loadModels();
  startWebcam();
  await setupCameras();
})

onBeforeUnmount(() => {
  stopWebcam()
})

const loadModels = async () => {
  const MODEL_URL = '/models'
  await faceapi.nets.tinyFaceDetector.loadFromUri(MODEL_URL)
  await faceapi.nets.faceLandmark68Net.loadFromUri(MODEL_URL)
  await faceapi.nets.faceRecognitionNet.loadFromUri(MODEL_URL)
  await faceapi.nets.faceExpressionNet.loadFromUri(MODEL_URL)
  isLoading.value = false
}

const startWebcam = async (videoElement, deviceId) => {
  navigator.mediaDevices
    .getUserMedia({ video: { deviceId } })
    .then((stream) => {
      if (video.value) {
        videoElement.srcObject = stream
      }
    })
    .catch((err) => console.error('Error accessing webcam:', err))
}

const getConnectedWebcams = async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  return devices.filter((device) => device.kind === "videoinput");
}

const setupCameras = async () => {
  const webcams =  await getConnectedWebcams();
  console.log("Webcams: ", webcams)
  if (webcams.length < 2) {
    alert("Two webcams are required!");
    return;
  }

  await startWebcam(video.value, webcams[0].deviceId);
  await startWebcam(video2.value, webcams[9].deviceId);
}

const stopWebcam = () => {
  const stream = video.value?.srcObject
  if (stream) {
    stream.getTracks().forEach((track) => track.stop())
  }
}

const detectFaces = async () => {
  const canvas = faceapi.createCanvasFromMedia(video.value)
  document.body.append(canvas)

  const displaySize = { width: video.value.width, height: video.value.height }
  faceapi.matchDimensions(canvas, displaySize)

  setInterval(async () => {
    const detections = await faceapi
      .detectAllFaces(video.value, new faceapi.TinyFaceDetectorOptions())
      .withFaceLandmarks()
      .withFaceExpressions()

    const resizedDetections = faceapi.resizeResults(detections, displaySize)
    canvas.getContext('2d').clearRect(0, 0, canvas.width, canvas.height)
    faceapi.draw.drawDetections(canvas, resizedDetections)
    faceapi.draw.drawFaceLandmarks(canvas, resizedDetections)
    faceapi.draw.drawFaceExpressions(canvas, resizedDetections)
  }, 100)
}
</script>

<template>
  <div class="flex flex-col items-center">
    <p v-if="isLoading" class="text-center">Loading Face Recognition Models...</p>
    <!-- <video ref="video" autoplay @play="detectFaces" class="border-2 border-gray-300 rounded"></video> -->

    <video ref="video" autoplay @play="detectFaces" class="w-1/2 border"></video>
    <video ref="video2" autoplay @play="detectFaces" class="w-1/2 border"></video>
  </div>
</template>

<style scoped>
video {
  width: 400px;
  height: 300px;
  border: 2px solid #000;
}
</style>
