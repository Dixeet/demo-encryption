<template>
  <TwoCols>
    <div class="mb-5">AES, RC4, DES, ChaCha20 etc...</div>
    <template #left>
      <div>
        <v-text-field label="secret" placeholder="secret" v-model="secret"></v-text-field>
        <v-text-field label="data" placeholder="data" v-model="data"></v-text-field>
        <div>
          <v-btn class="mr-1" variant="tonal" @click="encrypt">Chiffrer</v-btn>
          <v-btn variant="tonal" @click="decrypt">Déchiffrer</v-btn>
        </div>
      </div>
    </template>
    <template #right>
      <div>
        <v-row>
          <v-col cols="3">
            <div>Key :</div>
          </v-col>
          <v-col>
            {{ secret }}
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="3">
            <div>Donnée chiffrée :</div>
          </v-col>
          <v-col>
            {{ encrypted }}
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="3">
            <div>Donnée déchiffrée :</div>
          </v-col>
          <v-col>
            {{ decrypted }}
          </v-col>
        </v-row>
      </div>
    </template>
  </TwoCols>
</template>

<script setup>
import CryptoJS from 'crypto-js'
import TwoCols from '@/components/Layout/TwoCols.vue'
import { ref } from 'vue'

const secret = ref('my-secret')
const data = ref('foobar')

const encrypted = ref('')
const decrypted = ref('')

let enc = null

function encrypt() {
  enc = CryptoJS.AES.encrypt(data.value, secret.value)
  encrypted.value = enc.ciphertext.toString()
}

function decrypt() {
  const dec = CryptoJS.AES.decrypt(enc, secret.value)
  decrypted.value = dec.toString(CryptoJS.enc.Utf8) || 'impossible de déchiffrer'
}
</script>

<style lang="scss" scoped></style>
