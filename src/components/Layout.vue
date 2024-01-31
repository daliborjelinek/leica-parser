<template>
  <v-app id="inspire">
    <v-navigation-drawer v-model="store.drawer">
      <portal-target name="sidebar"></portal-target>
    </v-navigation-drawer>
    <v-app-bar>
      <v-app-bar-nav-icon icon="mdi-auto-fix" @click="store.drawer = !store.drawer"></v-app-bar-nav-icon>
      <v-app-bar-title>Leica parser</v-app-bar-title>
      <v-spacer />
      <portal-target name="header-button">
      </portal-target>
    </v-app-bar>
    <v-main>
      <home />
    </v-main>
  </v-app>
</template>

<script setup>
import { ref } from 'vue'

import { store } from '/src/state/state'


</script>
