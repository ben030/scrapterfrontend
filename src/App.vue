<template>
  <div
      class="bg-bg text-white text-xl h-screen flex justify-center"
      id="root"
  >
    <div
        class="px-2 sm:px-0 sm:w-2/3 flex flex-col child:flex-grow-0 h-full items-center"
    >
      <TitleComponent size="large">
        <template #title>Scrap<span class="text-blurplelight">ter</span></template>
        <template #subtitle>Sort your Flasks!</template>
      </TitleComponent>
      <div class="flex flex-col items-center bg-bglighter p-4 rounded mt-6 h-3/4 w-full">
        <CameraView class="h-1/2" @mouseenter="test = true"
                    @mouseleave="test = false"></CameraView>
        <PulseComponent></PulseComponent>
        <div class="h-1/2 w-full">
          <DetailView :product="test ? product : null" class="flex items-center"></DetailView>
        </div>
        <h1 ref="testText"></h1>
      </div>
    </div>
  </div>
  <canvas ref="videoScreenshot" class="hidden"></canvas>
</template>

<script setup lang="ts">
import CameraView from "./components/CameraView.vue";
import TitleComponent from "./components/TitleComponent.vue";
import DetailView from "./components/DetailView.vue";
import Product from "./models/product";
import {ProductType} from "./models/productType";
import {onBeforeUnmount, onMounted, ref} from "vue";
import PulseComponent from "./components/PulseComponent.vue";

const test = ref(false);
let videoPlayer: HTMLVideoElement | null;
let videoScreenshot = ref<HTMLCanvasElement | null>(null);
let testText = ref<HTMLParagraphElement>();

const product: Product = {
  name: "3 Käse Hoch",
  image: "https://de.openfoodfacts.org/images/products/305/764/025/7773/front_fr.265.400.jpg",
  productType: ProductType.GREEN
};

let connection: WebSocket | null = null;
let loopInterval: number | null = null;


onMounted(() => {
  videoPlayer = document.getElementById("videoPlayer") as HTMLVideoElement;
  connectWebsocket();

  //loopInterval = setInterval(loop, 1000);
  window.onclick = () => {
    loop();
  };
})

onBeforeUnmount(() => {
  closeWebsocket();

  if (loopInterval != null)
    clearInterval(loopInterval)
})

function connectWebsocket() {
  closeWebsocket();

  connection = new WebSocket("ws://172.20.10.4:8000/ws");

  connection.onopen = () => {
    console.info("WebSocket connected");
  };

  connection.onmessage = (d) => {

  };

  connection.onclose = () => {
    console.warn("WebSocket disconnected! Retrying")
    connectWebsocket();
  };

}

function closeWebsocket() {
  connection?.close();
}



function loop() {
  if (videoScreenshot.value != null && videoPlayer != null) {
    const ctx = videoScreenshot.value.getContext("2d")!;
    ctx.canvas.width = videoPlayer.videoWidth;
    ctx.canvas.height = videoPlayer.videoHeight;
    ctx.drawImage(videoPlayer, 0, 0, videoPlayer.videoWidth, videoPlayer.videoHeight);
    const base64 = videoScreenshot.value.toDataURL('image/jpeg');


    if (connection?.readyState == 1 ?? false)
      connection?.send(JSON.stringify({type: "image", data: {"base64": base64}}));

    ctx.clearRect(0, 0, videoPlayer.videoWidth, videoPlayer.videoHeight);
  }
}

</script>


<style scoped>

</style>
