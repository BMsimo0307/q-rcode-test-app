<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Mon Panier</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="cart.length === 0" class="text-center mt-10">
        <p class="text-gray-500">Aucun ticket en attente</p>
      </div>

      <ion-list v-else>
        <ion-item v-for="(item, index) in cart" :key="index">
          <ion-label>
            <h3>{{ item.event }}</h3>
            <p>{{ item.date }} – {{ item.location }}</p>
            <p class="text-xs text-gray-500">QR: {{ item.qr }}</p>
          </ion-label>
          <ion-button color="danger" fill="clear" @click="removeFromCart(index)">
            <ion-icon slot="icon-only" :icon="trashOutline" />
          </ion-button>
        </ion-item>
      </ion-list>

      <ion-button
        expand="block"
        class="ion-margin-top"
        :disabled="cart.length === 0"
        @click="confirmPurchase"
      >
        🎉 Valider l’achat
      </ion-button>

      <ion-toast
        :is-open="showToast"
        message="Tickets ajoutés à votre compte !"
        duration="2000"
        @didDismiss="showToast = false"
      />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonItem, IonLabel, IonButton, IonIcon, IonList, IonToast
} from '@ionic/vue'
import { ref } from 'vue'
import { trashOutline } from 'ionicons/icons'

const cart = ref(JSON.parse(localStorage.getItem('cart') || '[]'))
const showToast = ref(false)

function removeFromCart(index: number) {
  cart.value.splice(index, 1)
  localStorage.setItem('cart', JSON.stringify(cart.value))
}

function confirmPurchase() {
  const existing = JSON.parse(localStorage.getItem('tickets') || '[]')
  const updated = [...existing, ...cart.value]
  localStorage.setItem('tickets', JSON.stringify(updated))
  cart.value = []
  localStorage.removeItem('cart')
  showToast.value = true
}
</script>

<style scoped>
.text-center {
  text-align: center;
}
.mt-10 {
  margin-top: 10vh;
}
</style>
