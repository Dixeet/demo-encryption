<template>
  <TwoCols>
    <template #left>
      <div>
        <v-text-field label="data" placeholder="data" v-model="data"></v-text-field>
        <div>
          <v-btn class="mr-1" variant="tonal" @click="generate">Générer clés</v-btn>
          <v-btn class="mr-1" variant="tonal" @click="encrypt">Chiffrer</v-btn>
          <v-btn variant="tonal" @click="decrypt">Déchiffrer</v-btn>
        </div>
      </div>
    </template>
    <template #right>
      <div>
        <v-row>
          <v-col cols="3">
            <div>PublicKey :</div>
          </v-col>
          <v-col>
            <pre>
              {{ keys.publicKey }}
            </pre>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="3">
            <div>PrivateKey :</div>
          </v-col>
          <v-col>
            <pre>
              {{ keys.privateKey }}
            </pre>
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="3">
            <div>Donnée chiffrée avec la clé publique (décodé en texte) :</div>
          </v-col>
          <v-col>
            {{ encrypted }}
          </v-col>
        </v-row>
        <v-row>
          <v-col cols="3">
            <div>Donnée déchiffrée avec la clé privée (décodé en texte) :</div>
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
import rsa from 'js-crypto-rsa'
import TwoCols from '@/components/Layout/TwoCols.vue'
import { ref } from 'vue'

const data = ref('foobar')
const keys = ref({})
const encrypted = ref('')
const decrypted = ref('')

let enc = null

async function generate() {
  keys.value = await rsa.generateKey(2048)
}

async function encrypt() {
  const msg = new TextEncoder().encode(data.value)
  enc = await rsa.encrypt(msg, keys.value.publicKey)
  encrypted.value = new TextDecoder().decode(enc)
}

async function decrypt() {
  try {
    const res = await rsa.decrypt(enc, keys.value.privateKey)
    decrypted.value = new TextDecoder().decode(res)
  } catch (e) {
    decrypted.value = 'impossible de déchiffrer'
  }
}
</script>

<style lang="scss" scoped></style>
