<template>
  <ion-page>
    <ion-content class="ion-padding">
      <div class="text-center mt-10">
        <h2 class="text-2xl font-bold mb-6">Connexion</h2>

        <ion-item>
          <ion-label position="floating">Email</ion-label>
          <ion-input v-model="email" type="text" />
        </ion-item>

        <ion-item class="mt-4">
          <ion-label position="floating">Mot de passe</ion-label>
          <ion-input v-model="password" type="password" />
        </ion-item>

        <ion-button expand="block" class="mt-6" @click="login">Se connecter</ion-button>

        <p v-if="errorMessage" class="text-red-500 mt-4 text-sm">{{ errorMessage }}</p>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage,
  IonContent,
  IonItem,
  IonLabel,
  IonInput,
  IonButton
} from '@ionic/vue'
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const email = ref('')
const password = ref('')
const errorMessage = ref('')
const router = useRouter()

function login() {
  if (email.value === 'admin' && password.value === 'admin') {
    localStorage.setItem('auth', 'true')
    localStorage.setItem('role', 'admin')
    localStorage.setItem('name', 'Administrateur')
    router.push('/tabs/tab1')
  } else {
    errorMessage.value = 'Identifiants incorrects.'
  }
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
