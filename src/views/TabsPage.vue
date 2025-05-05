<template>
  <ion-page>
    <ion-tabs>
      <ion-router-outlet></ion-router-outlet>

      <ion-tab-bar slot="bottom">
        <ion-tab-button tab="tab1" href="/tabs/tab1">
          <ion-icon :icon="qrCodeOutline" />
          <ion-label>Accueil</ion-label>
        </ion-tab-button>

        <ion-tab-button tab="tab2" href="/tabs/tab2">
          <ion-icon :icon="barChartOutline" />
          <ion-label>Stats</ion-label>
        </ion-tab-button>
        
        <ion-tab-button tab="tab3" href="/tabs/tab3">
          <ion-icon :icon="scanOutline" />
          <ion-label>Scanner</ion-label>
        </ion-tab-button>
        <ion-tab-button tab="tab4" href="/tabs/tab4">
          <ion-icon :icon="analyticsOutline" />
          <ion-label>Dashboard</ion-label>
        </ion-tab-button>

      </ion-tab-bar>
    </ion-tabs>
  </ion-page>
</template>

<script setup lang="ts">
import {
  IonTabBar, IonTabButton, IonTabs, IonLabel,
  IonIcon, IonPage, IonRouterOutlet
} from '@ionic/vue'

import {
  qrCodeOutline,
  scanOutline,
  barChartOutline
} from 'ionicons/icons'
</script>
