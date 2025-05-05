<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Scanner QR Code (Web)</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="getRole() !== 'admin'" class="text-center mt-10">
        <p class="text-red-600 text-lg font-semibold">
          Accès réservé aux organisateurs
        </p>
      </div>

      <div v-else>
        <video ref="videoRef" width="100%" autoplay muted playsinline></video>

        <ion-card v-if="scanResult" class="ion-margin-top">
          <ion-card-header>
            <ion-card-title>✅ QR détecté</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <p><strong>Contenu :</strong> {{ scanResult }}</p>
          </ion-card-content>
        </ion-card>

        <ion-text color="danger" class="ion-text-center mt-4" v-if="error">
          {{ error }}
        </ion-text>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonHeader,
  IonToolbar,
  IonTitle,
  IonContent,
  IonCard,
  IonCardHeader,
  IonCardTitle,
  IonCardContent,
  IonText
} from '@ionic/vue'
import { ref, onMounted, onBeforeUnmount } from 'vue'
import { BrowserMultiFormatReader } from '@zxing/browser'

const videoRef = ref<HTMLVideoElement | null>(null)
const scanResult = ref('')
const error = ref('')
let codeReader: BrowserMultiFormatReader | null = null

function getRole() {
  return localStorage.getItem('role')
}

onMounted(() => {
  codeReader = new BrowserMultiFormatReader()
  codeReader.decodeFromVideoDevice(undefined, videoRef.value!, (result, err) => {
    if (result) {
      scanResult.value = result.getText()
    }
    if (err && !(err instanceof DOMException)) {
      error.value = 'Erreur : ' + err
    }
  }).catch((err) => {
    error.value = 'Accès à la caméra refusé'
  })
})

onBeforeUnmount(() => {
  if (codeReader) {
    codeReader.reset()
  }
})
</script>

<style scoped>
video {
  border: 1px solid #ccc;
  border-radius: 10px;
  margin-top: 10px;
}
.mt-10 {
  margin-top: 10vh;
}
</style>
