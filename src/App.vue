<template>
  <suspense>
    <v-layout class="">
      <v-main class="" style="min-height: 300px">
        <v-tabs bg-color="primary" v-model="tab">
          <v-tab value="symetric">Chiffrement symétrique</v-tab>
          <v-tab value="asymetric">Chiffrement asymétrique</v-tab>
          <v-tab value="tls">SSL/TLS</v-tab>
        </v-tabs>

        <v-window v-model="tab">
          <v-window-item value="symetric"><Symetric /></v-window-item>

          <v-window-item value="asymetric"><Asymetric /></v-window-item>

          <v-window-item value="tls"><Tls /></v-window-item>
        </v-window>
      </v-main>
    </v-layout>
  </suspense>
</template>

<script setup>
import Symetric from '@/components/Symetric.vue'
import Asymetric from '@/components/Asymetric.vue'
import Tls from '@/components/Tls.vue'
import { ref } from 'vue'

const tab = ref('symetric')
</script>

<style scoped></style>
