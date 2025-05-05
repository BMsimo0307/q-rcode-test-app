<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Acheter un ticket</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <ion-card v-for="fest in festivals" :key="fest.id" class="ion-margin-bottom">
        <ion-card-header>
          <ion-card-title>{{ fest.name }}</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <p>📍 {{ fest.location }}</p>
          <p>📅 {{ fest.date }}</p>
          <ion-button expand="block" @click="buyTicket(fest)">Acheter ce ticket</ion-button>
        </ion-card-content>
      </ion-card>

      <ion-toast
        :is-open="showToast"
        message="Ticket ajouté à votre compte ✅"
        duration="2000"
        @didDismiss="showToast = false"
      />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonCard, IonCardHeader, IonCardTitle, IonCardContent,
  IonButton, IonToast
} from '@ionic/vue'
import { ref } from 'vue'

const festivals = [
  { id: 1, name: 'Festival A', date: '2025-06-01', location: 'Toulouse' },
  { id: 2, name: 'Festival B', date: '2025-07-15', location: 'Paris' },
  { id: 3, name: 'Festival C', date: '2025-08-20', location: 'Lyon' }
]

const showToast = ref(false)

function buyTicket(festival: any) {
  const user = localStorage.getItem('name') || 'Anonyme'
  const qrCode = `${festival.name.replace(/\s/g, '-')}-${user}-${Date.now()}`

  const newTicket = {
    event: festival.name,
    date: festival.date,
    location: festival.location,
    qr: qrCode
  }

    const existingCart = JSON.parse(localStorage.getItem('cart') || '[]')
    existingCart.push(newTicket)
    localStorage.setItem('cart', JSON.stringify(existingCart))


  showToast.value = true
}
</script>
