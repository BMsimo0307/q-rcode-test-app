<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Admin - Accueil</ion-title>
        <ion-buttons slot="end">
          <ion-button @click="confirmLogout">
            <ion-icon :icon="logOutOutline" />
          </ion-button>
        </ion-buttons>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding ion-text-center">
      <div class="mt-6">
        <h2 class="text-xl font-semibold">Bienvenue {{ name }}</h2>
        <p class="text-gray-600 mb-4">
          Vous êtes connecté en tant qu'organisateur.
        </p>
      </div>

      <ion-card v-if="total > 0" class="ion-margin-top">
        <ion-card-header>
          <ion-card-title>📊 Statut des Participants</ion-card-title>
        </ion-card-header>
        <ion-card-content>
          <p>👥 Total : <strong>{{ total }}</strong></p>
          <p>✅ Scannés : <strong class="text-green">{{ scanned }}</strong></p>
          <p>❌ Non scannés : <strong class="text-red">{{ notScanned }}</strong></p>

          <ion-button
            color="warning"
            expand="block"
            class="mt-4"
            @click="resetStatuses"
          >
            🔄 Réinitialiser les statuts
          </ion-button>
        </ion-card-content>
      </ion-card>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonButtons, IonButton, IonIcon,
  IonCard, IonCardHeader, IonCardTitle, IonCardContent,
  alertController
} from '@ionic/vue'
import { useRouter } from 'vue-router'
import { logOutOutline } from 'ionicons/icons'
import { ref, computed } from 'vue'

const router = useRouter()
const name = ref(localStorage.getItem('name') || 'Administrateur')

const participants = ref<any[]>(JSON.parse(localStorage.getItem('participants') || '[]'))

const total = computed(() => participants.value.length)
const scanned = computed(() => participants.value.filter(p => p.status === 'scanned').length)
const notScanned = computed(() => participants.value.filter(p => p.status !== 'scanned').length)

function resetStatuses() {
  participants.value.forEach(p => (p.status = 'not scanned'))
  localStorage.setItem('participants', JSON.stringify(participants.value))
}

async function confirmLogout() {
  const alert = await alertController.create({
    header: 'Déconnexion',
    message: 'Voulez-vous vraiment vous déconnecter ?',
    buttons: [
      { text: 'Annuler', role: 'cancel' },
      {
        text: 'Se déconnecter',
        handler: () => {
          localStorage.clear()
          router.replace('/login')
        }
      }
    ]
  })

  await alert.present()
}
</script>

<style scoped>
.mt-6 {
  margin-top: 6vh;
}
.text-red {
  color: #eb445a;
}
.text-green {
  color: #2dd36f;
}
</style>
