<template>
  <ThreeCols>
    <v-btn class="mb-5" variant="tonal" @click="nextStep">{{ btnTxt }}</v-btn>
    <template #left>
      <div>
        <h3>Client</h3>
        <v-divider class="my-2" />
      </div>
      <pre>{{ clientTxt }}</pre>
    </template>
    <template #center>
      <div>
        <h3>{{ msgFrom }}</h3>
        <v-divider class="my-2" />
      </div>
      <pre>{{ msgTxt }}</pre>
    </template>
    <template #right>
      <div>
        <h3>Server</h3>
        <v-divider class="my-2" />
      </div>
      <pre>{{ serverTxt }}</pre>
    </template>
  </ThreeCols>
</template>

<script setup>
import CryptoJS from 'crypto-js'
import ThreeCols from '@/components/Layout/ThreeCols.vue'
import { ref } from 'vue'
import rsa from 'js-crypto-rsa'

const baseMsgFrom = 'Message échangé'
const clientMsgFrom = 'Message du client ⮞⮞⮞⮞⮞⮞'
const serverMsgFrom = '⮜⮜⮜⮜⮜⮜ Message du server'

let asymKeys = await rsa.generateKey(2048)
let nextStepName = 'baseStep'
let enc = null
let premaster = ''
let sessionKey = ''
const steps = {
  baseStep: baseStep,
  clientHello: clientHello,
  serverHello: serverHello,
  authentication: authentication,
  premasterSecret: premasterSecret,
  privateKeyUsed: privateKeyUsed,
  sessionKeys: sessionKeys,
  clientReady: clientReady,
  serverReady: serverReady,
  exchangeEncryptedData: exchangeEncryptedData
}

const msgFrom = ref(baseMsgFrom)
const clientTxt = ref('')
const serverTxt = ref('')
const msgTxt = ref('')
const btnTxt = ref('')

nextStep()

function nextStep() {
  steps[nextStepName]()
}

function baseStep() {
  btnTxt.value = 'Initialiser le handshake TLS'
  nextStepName = 'clientHello'
}

function clientHello() {
  msgFrom.value = clientMsgFrom
  clientTxt.value = `Initialise le handshake en envoyant un "hello'.

Le message inclus les versions TLS et les suites de chiffrement supportées`
  msgTxt.value = {
    msg: 'hello',
    tlsAccepted: [1.2, 1.3],
    cipherSuitesAccepted: ['A', 'B', 'D']
  }
  btnTxt.value = 'Répondre à la demande de handshake'
  nextStepName = 'serverHello'
}

function serverHello() {
  msgFrom.value = serverMsgFrom
  clientTxt.value = ''
  serverTxt.value = `Demande de handshake reçu

Répondre avec le certificat SSL contenant la clé publique, l'authorité émettrice du certificat, et la suite de chiffrement choisie`
  msgTxt.value = {
    cipherSuite: 'B',
    tls: 1.2,
    sslCertif: {
      publicKey: asymKeys.publicKey,
      authIssuer: 'Cloudflare'
    }
  }
  btnTxt.value = "Vérifier l'authentification"
  nextStepName = 'authentication'
}

function authentication() {
  msgFrom.value = baseMsgFrom
  clientTxt.value =
    "Vérification auprès de l'authorité de l'identité émettrice et de l'authenticité du server"
  serverTxt.value = ''
  msgTxt.value = ''
  btnTxt.value = 'Générer et envoyer la clé premaster'
  nextStepName = 'premasterSecret'
}

async function premasterSecret() {
  msgFrom.value = clientMsgFrom
  premaster = generateRandomString()
  clientTxt.value = `Authenticité vérifié

Génération aléatoire, chiffrement avec la clé publique et envoie d'une premaster secret

premaster key: ${premaster}`
  enc = await rsa.encrypt(new TextEncoder().encode(premaster), asymKeys.publicKey)
  serverTxt.value = ''
  msgTxt.value = {
    premasterSecret: enc
  }
  btnTxt.value = 'Déchiffrer la premaster'
  nextStepName = 'privateKeyUsed'
}

async function privateKeyUsed() {
  msgFrom.value = baseMsgFrom
  clientTxt.value = `premaster key: ${premaster}`
  const dec = await rsa.decrypt(enc, asymKeys.privateKey)
  serverTxt.value = `Déchiffrer la premaster avec la clé privée

premaster secret : ${new TextDecoder().decode(dec)}`
  msgTxt.value = ''
  btnTxt.value = 'Générer clé de session'
  nextStepName = 'sessionKeys'
}

function sessionKeys() {
  msgFrom.value = baseMsgFrom
  sessionKey = generateRandomString() + generateRandomString() + generateRandomString()
  clientTxt.value = `Génération de la clé de session à partir de la premaster secret

clé de session :
${sessionKey}`
  serverTxt.value = `Génération de la clé de session à partir de la premaster secret

clé de session :
${sessionKey}`
  msgTxt.value = ''
  btnTxt.value = 'Client prêt'
  nextStepName = 'clientReady'
}

function clientReady() {
  msgFrom.value = clientMsgFrom
  clientTxt.value = "Signalé au server d'être prêt à communiquer de manière chiffrée"
  serverTxt.value = ''
  msgTxt.value = {
    msg: 'ready'
  }
  btnTxt.value = 'Serveur prêt'
  nextStepName = 'serverReady'
}

function serverReady() {
  msgFrom.value = serverMsgFrom
  clientTxt.value = ''
  serverTxt.value = `Client prêt.

Répondre que le serveur l'est aussi`
  msgTxt.value = {
    msg: 'ready'
  }
  btnTxt.value = 'Echanger de la donnée de manière sécurisée'
  nextStepName = 'exchangeEncryptedData'
}

function exchangeEncryptedData() {
  msgFrom.value = clientMsgFrom
  const data = {
    foo: 'bar'
  }
  const encData = CryptoJS.AES.encrypt(JSON.stringify(data), sessionKey)
  clientTxt.value = `Envoyer de la donnée chiffrée au server

clé de session : ${sessionKey}

data :
${JSON.stringify(data, null, 4)}`

  const dec = CryptoJS.AES.decrypt(encData, sessionKey)
  serverTxt.value = `Donnée chiffrée reçu

clé de session : ${sessionKey}

data :
${JSON.stringify(dec.toString(CryptoJS.enc.Utf8), null, 4)}`
  msgTxt.value = encData.ciphertext.toString()
  btnTxt.value = 'Fin'
}

function generateRandomString() {
  return Math.floor(Math.random() * Date.now()).toString(36)
}
</script>

<style scoped>
pre {
  white-space: pre-wrap;
  text-align: left;
}
</style>
