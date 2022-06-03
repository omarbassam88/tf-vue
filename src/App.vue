<script setup>
import { ref } from "vue";
import "@tensorflow/tfjs-backend-cpu";
import "@tensorflow/tfjs-backend-webgl";
import * as cocoSsd from "@tensorflow-models/coco-ssd";

const imgRef = ref(null);
const videoRef = ref(null);
const results = ref([]);
const isLoading = ref(false);
const isStreaming = ref(false);

const placeholderImage = "https://images.unsplash.com/photo-1533022139390-e31c488d69e2?ixlib=rb-1.2.1&ixid=eyJhcHBfaWQiOjEyMDd9&auto=format&fit=crop&w=2089&q=80"


const detect = async () => {
    const img = imgRef.value;
    isLoading.value = true;
    // Load the model.
    const model = await cocoSsd.load();
    // Classify the image.
    const predictions = await model.detect(img);
    results.value = predictions;
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
    console.log("Stopping Streaming");
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
    <section class="w-full">
        <h1 class="text-3xl font-bold">TensorFlow Object Detector</h1>
        <!-- Camera Controls -->
        <div class="flex justify-center">
            <button @click="isStreaming ? stopStreaming : openCamera"
                class="btn text-white w-32 mx-1 my-4 bg-gradient-to-r from-blue-800 to-indigo-800">
                {{ isStreaming ? "Close Camera" : "Open Camera" }}
            </button>
            <button v-if="isStreaming" @click="snapshot"
                class="btn text-white w-32 mx-1 my-4 bg-gradient-to-r from-blue-800 to-indigo-800">
                Take a snapshot
            </button>
        </div>
        <!-- Camera Video or Captured image -->
        <div class="flex p-5 justify-center">
            <video v-show="isStreaming" ref="videoRef" width="100" autoplay="true"></video>
            <img v-show="!isStreaming" ref="imgRef" :src="placeholderImage" width="500" crossorigin="anonymous" />
        </div>
        <!-- Detect Object Button -->
        <button @click="detect" class="btn text-white w-32 mx-1 my-4 bg-gradient-to-r from-red-800 to-pink-600">
            {{ isLoading ? "Loading..." : "Detect Object" }}
        </button>
        <!-- Results Section -->
        <div v-if="results.length > 0" class="flex gap-2 justify-center">
            <a v-for="item in results" :key="item.id" :href="`http://google.com/search?q=${item.class}`" target="_blank"
                class="text-white bg-blue-600 p-3 rounded-xl">
                {{ item.class }}
            </a>
        </div>
    </section>
</template>

<style>
body {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    margin-top: 60px;
}

.btn {
    @apply p-3 rounded-lg shadow-md;
}
</style>
