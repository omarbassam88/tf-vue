<script setup>
import { ref } from "vue";
import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";
import * as cocoSsd from "@tensorflow-models/coco-ssd";

console.log(cocoSsd);

const imgRef = ref("");
const videoRef = ref("");
const result = ref("");
const isLoading = ref(false);
const isStreaming = ref(false);
const detect = async () => {
  const img = imgRef.value;
  isLoading.value = true;
  // Load the model.
  const model = await cocoSsd.load();
  // Classify the image.
  const predictions = await model.detect(img);
  result.value = predictions;
  isLoading.value = false;
  console.log("Predictions: ");
  console.log(predictions);
};
const openCamera = async () => {
  const devices = await navigator.mediaDevices.enumerateDevices();
  const cams = devices.filter((device) => device.kind == "videoinput");
  const camId = cams[0].deviceId;
  if (navigator.mediaDevices.getUserMedia) {
    navigator.mediaDevices
      .getUserMedia({ video: { deviceId: { exact: camId } } })
      .then((stream) => {
        videoRef.value.srcObject = stream;
      });
  }
  isStreaming.value = true;
};
const stopStreaming = () => {
  const stream = videoRef.value.srcObject;
  const tracks = stream.getTracks();
  tracks.map((track) => track.stop());
  isStreaming.value = false;
};
const snapshot = () => {
  const canvas = document.createElement("canvas");
  const ctx = canvas.getContext("2d");
  ctx.drawImage(videoRef.value, 0, 0, 200, 200);
  const data = canvas.toDataURL("image/png");
  isStreaming.value = false;
  imgRef.value.setAttribute("src", data);
};
</script>

<template>
  <section class="flex w-full">
    <div class="m-auto">
      <div class="m-10">
        <h1 class="text-3xl font-bold">TensorFlow Object Detection</h1>
        <button
          v-if="!isStreaming"
          @click="openCamera"
          class="text-white m-4 p-3 w-32 mx-1 my-4 rounded-lg shadow-md bg-gradient-to-r from-blue-800 to-indigo-800"
        >
          Open Camera
        </button>

        <div v-else class="flex justify-between">
          <button
            @click="stopStreaming"
            class="text-white m-4 p-3 w-32 mx-1 my-4 rounded-lg shadow-md bg-gradient-to-r from-blue-800 to-indigo-800"
          >
            Open Camera
          </button>
          <button
            @click="snapshot"
            class="text-white m-4 p-3 w-32 mx-1 my-4 rounded-lg shadow-md bg-gradient-to-r from-blue-800 to-indigo-800"
          >
            Take a snapshot
          </button>
        </div>

        <video
          v-if="isStreaming"
          ref="videoRef"
          width="100"
          autoplay="true"
        ></video>
        <div v-else class="flex justify-center">
          <img
            ref="imgRef"
            src="https://images.unsplash.com/photo-1533022139390-e31c488d69e2?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2089&q=80"
            width="500"
            crossorigin="anonymous"
          />
        </div>
        <button
          @click="detect"
          class="text-white m-4 p-3 w-32 mx-1 rounded-lg shadow-md bg-gradient-to-r from-red-800 to-pink-600"
        >
          <span v-if="isLoading"> Loading...</span>
          <span v-else> Detect Object</span>
        </button>
        <div v-if="result.length > 0" class="text-2xl my-2">
          <p v-for="item in result" :key="item.id">
            {{ item.class }}
          </p>
        </div>
      </div>
    </div>
  </section>
</template>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
