<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Dashboard</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="getRole() !== 'admin'" class="text-center mt-10">
        <p class="text-red-600 text-lg font-semibold">Accès réservé aux organisateurs</p>
      </div>

      <div v-else>
        <h2 class="text-lg font-semibold mb-4">Statistiques générales</h2>

        <ion-card>
          <ion-card-content>
            <p>Total de participants : <strong>{{ total }}</strong></p>
            <p>Scannés : <strong class="text-green">{{ scanned }}</strong></p>
            <p>Non scannés : <strong class="text-yellow">{{ notScanned }}</strong></p>
            <p>Invalides : <strong class="text-red">{{ invalid }}</strong></p>
          </ion-card-content>
        </ion-card>

      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonCard, IonCardContent
} from '@ionic/vue'
import { ref, computed, onMounted } from 'vue'

const participants = ref<any[]>([])

const scanned = computed(() => participants.value.filter(p => p.status === 'scanned').length)
const notScanned = computed(() => participants.value.filter(p => p.status === 'not scanned').length)
const invalid = computed(() => participants.value.filter(p => p.status === 'invalid').length)
const total = computed(() => participants.value.length)

function getRole() {
  return localStorage.getItem('role')
}

onMounted(() => {
  const saved = localStorage.getItem('participants')
  if (saved) participants.value = JSON.parse(saved)
})
</script>

<style scoped>
.text-green {
  color: #2dd36f;
}
.text-yellow {
  color: #ffc409;
}
.text-red {
  color: #eb445a;
}
.mt-10 {
  margin-top: 10vh;
}
</style>
