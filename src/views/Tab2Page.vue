<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Scanner un QR Code</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content class="ion-padding">
      <div v-if="getRole() !== 'admin'" class="text-center mt-10">
        <p class="text-red-600 text-lg font-semibold">Accès réservé aux organisateurs</p>
      </div>

      <div v-else>
        <ion-item>
          <ion-label>Importer les participants (.csv)</ion-label>
          <input type="file" accept=".csv" @change="handleCSV" />
        </ion-item>

        <ion-button expand="block" class="ion-margin-top" @click="exportCSV">
          ⬇️ Exporter les participants
        </ion-button>

        <ion-card class="ion-margin-top">
          <ion-card-header>
            <ion-card-title>📊 Statistiques</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <p>Total : <strong>{{ stats.total }}</strong></p>
            <p>🎯 Scannés : <strong class="text-green">{{ stats.scanned }}</strong></p>
            <p>🕒 Non scannés : <strong class="text-warning">{{ stats.notScanned }}</strong></p>
            <p>❌ Invalides : <strong class="text-red">{{ stats.invalid }}</strong></p>
          </ion-card-content>
        </ion-card>

        <ion-card class="ion-margin-top">
          <ion-card-header>
            <ion-card-title>➕ Ajouter un participant</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <ion-item>
              <ion-label position="stacked">Nom</ion-label>
              <ion-input v-model="newParticipant.name" placeholder="Nom complet" />
            </ion-item>

            <ion-item>
              <ion-label position="stacked">Événement</ion-label>
              <ion-input v-model="newParticipant.event" placeholder="Nom du festival" />
            </ion-item>

            <ion-item>
              <ion-label position="stacked">Type</ion-label>
              <ion-select v-model="newParticipant.type" placeholder="Choisir un type">
                <ion-select-option value="normal">Normal</ion-select-option>
                <ion-select-option value="guest">Guest</ion-select-option>
                <ion-select-option value="manual">Invitation</ion-select-option>
              </ion-select>
            </ion-item>

            <ion-button expand="block" class="mt-4" @click="addManualParticipant">
              Ajouter
            </ion-button>
          </ion-card-content>
        </ion-card>

        <ion-item v-if="allEvents.length > 0" class="ion-margin-top">
          <ion-label>Filtrer par festival :</ion-label>
          <ion-select v-model="selectedEvent" placeholder="Choisir un festival">
            <ion-select-option v-for="event in allEvents" :key="event" :value="event">
              {{ event }}
            </ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item v-if="filteredParticipants.length > 0" class="ion-margin-top">
          <ion-label position="stacked">Rechercher un participant :</ion-label>
          <ion-input v-model="searchTerm" placeholder="Tapez un nom..." />
        </ion-item>

        <!-- Scanner -->
        <qrcode-stream @decode="handleScan" @init="onInit" />

        <!-- Résultat du scan -->
        <ion-card v-if="scanResult" class="ion-margin-top">
          <ion-card-header>
            <ion-card-title>🎉 Accès autorisé</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <p><strong>Nom :</strong> {{ scanResult.name }}</p>
            <p><strong>Événement :</strong> {{ scanResult.event }}</p>
            <p><strong>Type :</strong> {{ scanResult.type }}</p>
            <p><strong>Status :</strong> {{ scanResult.status }}</p>
          </ion-card-content>
        </ion-card>

        <ion-text color="danger" class="ion-text-center mt-4" v-if="errorMessage">
          {{ errorMessage }}
        </ion-text>

        <ion-card class="ion-margin-top" v-if="visibleParticipants.length > 0">
          <ion-card-header>
            <ion-card-title>Participants - {{ selectedEvent }}</ion-card-title>
          </ion-card-header>
          <ion-card-content>
            <ion-list>
              <ion-item v-for="(p, index) in visibleParticipants" :key="index">
                <ion-label>
                  <h3>
                    {{ p.name }}
                    <span class="badge" :class="typeClass(p.type)">({{ p.type }})</span>
                  </h3>
                  <p>🔑 QR: {{ p.qr }}</p>
                  <p>Status:
                    <strong :class="p.status === 'scanned' ? 'text-green' : p.status === 'invalid' ? 'text-red' : 'text-warning'">
                      {{ p.status }}
                    </strong>
                  </p>
                </ion-label>
                <ion-select
                  interface="popover"
                  :value="p.status"
                  @ionChange="updateStatus(index, $event.detail.value)"
                >
                  <ion-select-option value="not scanned">not scanned</ion-select-option>
                  <ion-select-option value="scanned">scanned</ion-select-option>
                  <ion-select-option value="invalid">invalid</ion-select-option>
                </ion-select>
              </ion-item>
            </ion-list>
          </ion-card-content>
        </ion-card>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonPage, IonHeader, IonToolbar, IonTitle, IonContent,
  IonLabel, IonItem, IonCard, IonCardHeader, IonCardTitle,
  IonCardContent, IonText, IonList, IonSelect, IonSelectOption,
  IonInput, IonButton
} from '@ionic/vue'
import { ref, computed, onMounted, watch } from 'vue'
import Papa from 'papaparse'

const participants = ref<any[]>([])
const scanResult = ref<any | null>(null)
const errorMessage = ref<string | null>(null)

const allEvents = ref<string[]>([])
const selectedEvent = ref<string | null>(null)
const searchTerm = ref<string>('')

const newParticipant = ref({
  name: '',
  event: '',
  type: 'normal'
})

onMounted(() => {
  const saved = localStorage.getItem('participants')
  if (saved) {
    const parsed = JSON.parse(saved)
    participants.value = parsed.map((p: any) => ({
      ...p,
      status: p.status || 'not scanned',
      type: p.type || 'normal'
    }))
    allEvents.value = [...new Set(participants.value.map((p: any) => p.event))]
    selectedEvent.value = allEvents.value[0] || null
  }
})

watch(participants, () => {
  localStorage.setItem('participants', JSON.stringify(participants.value))
}, { deep: true })

function getRole() {
  return localStorage.getItem('role')
}

function handleCSV(event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0]
  if (file) {
    Papa.parse(file, {
      header: true,
      skipEmptyLines: true,
      complete: result => {
        const parsed = result.data.map((p: any) => ({
          ...p,
          status: p.status || 'not scanned',
          type: p.type || 'normal'
        }))
        participants.value = parsed
        localStorage.setItem('participants', JSON.stringify(participants.value))
        allEvents.value = [...new Set(participants.value.map((p: any) => p.event))]
        selectedEvent.value = allEvents.value[0] || null
      }
    })
  }
}

function addManualParticipant() {
  if (!newParticipant.value.name || !newParticipant.value.event) return
  const qr = `${newParticipant.value.event.replace(/\s/g, '-')}-${Date.now()}`
  participants.value.push({
    name: newParticipant.value.name,
    event: newParticipant.value.event,
    type: newParticipant.value.type,
    qr,
    status: 'not scanned'
  })
  allEvents.value = [...new Set(participants.value.map(p => p.event))]
  newParticipant.value = { name: '', event: '', type: 'normal' }
}

function handleScan(value: string) {
  const found = participants.value.find(p => p.qr === value)
  if (!found) {
    scanResult.value = {
      name: 'Inconnu',
      event: 'Non trouvé',
      type: 'N/A',
      status: 'invalid'
    }
    errorMessage.value = 'QR Code invalide ❌'
    return
  }

  if (found.status === 'scanned') {
    scanResult.value = null
    errorMessage.value = 'QR déjà utilisé 🚫'
    return
  }

  found.status = 'scanned'
  scanResult.value = found
  errorMessage.value = null
}

function exportCSV() {
  const csv = Papa.unparse(participants.value)
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  const url = URL.createObjectURL(blob)
  link.setAttribute('href', url)
  link.setAttribute('download', 'participants_export.csv')
  document.body.appendChild(link)
  link.click()
  document.body.removeChild(link)
}

function updateStatus(index: number, newStatus: string) {
  participants.value[index].status = newStatus
  localStorage.setItem('participants', JSON.stringify(participants.value))
}

const filteredParticipants = computed(() =>
  selectedEvent.value
    ? participants.value.filter(p => p.event === selectedEvent.value)
    : []
)

const visibleParticipants = computed(() =>
  filteredParticipants.value.filter(p =>
    p.name.toLowerCase().includes(searchTerm.value.toLowerCase())
  )
)

const stats = computed(() => {
  const total = participants.value.length
  const scanned = participants.value.filter(p => p.status === 'scanned').length
  const invalid = participants.value.filter(p => p.status === 'invalid').length
  const notScanned = total - scanned - invalid
  return { total, scanned, notScanned, invalid }
})

function onInit(promise: any) {
  promise.catch(() => {
    errorMessage.value = 'Erreur d’accès à la caméra'
  })
}

function typeClass(type: string) {
  if (type === 'guest') return 'guest'
  if (type === 'manual') return 'manual'
  return 'normal'
}
</script>

<style scoped>
input[type="file"] {
  margin-top: 10px;
}
.text-red {
  color: #eb445a;
}
.text-green {
  color: #2dd36f;
}
.text-warning {
  color: #ffc409;
}
.badge {
  font-size: 12px;
  padding: 2px 6px;
  border-radius: 10px;
  margin-left: 6px;
  text-transform: uppercase;
}
.guest {
  background: #ffce00;
  color: #000;
}
.manual {
  background: #3dc2ff;
  color: white;
}
.normal {
  background: #e0e0e0;
  color: #333;
}
</style>
