<script setup>
import { computed, onMounted, ref } from 'vue'
import QRCode from 'qrcode'

const adjectives = ['bouncy', 'cheerful', 'curious', 'dapper', 'fuzzy', 'jolly', 'merry', 'peppy', 'sparkly', 'sunny', 'wobbly']
const animals = ['alpaca', 'badger', 'beaver', 'capybara', 'dolphin', 'hedgehog', 'lemur', 'otter', 'panda', 'penguin', 'puffin', 'raccoon']
const pick = (items) => items[Math.floor(Math.random() * items.length)]
const text = ref(`https://www.${pick(adjectives)}${pick(animals)}.club/`)
const qrText = ref('')
const blackTransparent = ref('')
const blackWhite = ref('')
const whiteTransparent = ref('')
const isDarkPreview = ref(false)
const copyLabel = ref('Copy text')

const fileSafeName = computed(() => text.value.trim().replace(/[^a-z0-9]+/gi, '-').replace(/^-|-$/g, '').slice(0, 32) || 'qr-code')

function makeFixedWidthText(value) {
  const code = QRCode.create(value || ' ', { errorCorrectionLevel: 'M' })
  const { data, size } = code.modules
  const quietZone = 2
  const rows = []
  for (let y = -quietZone; y < size + quietZone; y += 1) {
    let row = ''
    for (let x = -quietZone; x < size + quietZone; x += 1) {
      const dark = x >= 0 && x < size && y >= 0 && y < size && data[y * size + x]
      row += dark ? '██' : '  '
    }
    rows.push(row)
  }
  return rows.join('\n')
}

async function renderCode() {
  const value = text.value || ' '
  qrText.value = makeFixedWidthText(value)
  const baseOptions = { errorCorrectionLevel: 'M', margin: 2, width: 800 }
  const [transparent, white, inverted] = await Promise.all([
    QRCode.toDataURL(value, { ...baseOptions, color: { dark: '#000000ff', light: '#00000000' } }),
    QRCode.toDataURL(value, { ...baseOptions, color: { dark: '#000000ff', light: '#ffffffff' } }),
    QRCode.toDataURL(value, { ...baseOptions, color: { dark: '#ffffffff', light: '#00000000' } }),
  ])
  if (value === (text.value || ' ')) {
    blackTransparent.value = transparent
    blackWhite.value = white
    whiteTransparent.value = inverted
  }
}

async function copyText() {
  await navigator.clipboard.writeText(qrText.value)
  copyLabel.value = 'Copied!'
  window.setTimeout(() => { copyLabel.value = 'Copy text' }, 1500)
}

onMounted(renderCode)
</script>

<template>
  <main class="app-shell">
    <header>
      <h1 aria-label="QR CODE"><span aria-hidden="true">Q</span><span aria-hidden="true">R</span><span aria-hidden="true"></span><span aria-hidden="true">C</span><span aria-hidden="true">O</span><span aria-hidden="true">D</span><span aria-hidden="true">E</span></h1>
    </header>
    <section class="input-card" aria-labelledby="text-label">
      <div class="field-heading"><label id="text-label" for="text-input">Text to encode</label><span>{{ text.length }} characters</span></div>
      <input id="text-input" v-model="text" type="text" placeholder="Enter text, a URL, or anything else" @keyup="renderCode" />
    </section>
    <section class="outputs" aria-label="QR code outputs">
      <article class="output-card text-card">
        <div class="card-heading"><div><p class="format">Plain text</p><h2>Fixed-width text</h2></div><button class="copy-button" type="button" @click="copyText">{{ copyLabel }}</button></div>
        <textarea class="qr-text" :value="qrText" aria-label="Copyable fixed-width QR text" readonly />
      </article>
      <article class="output-card">
        <div class="card-heading"><div><p class="format">PNG · transparent</p><h2>Black on transparent</h2></div><a :href="blackTransparent" :download="`${fileSafeName}-black-transparent.png`">Download</a></div>
        <div class="image-frame checkerboard"><img :src="blackTransparent" alt="Black QR code on transparent background" /></div>
      </article>
      <article class="output-card">
        <div class="card-heading"><div><p class="format">PNG · opaque</p><h2>Black on white</h2></div><a :href="blackWhite" :download="`${fileSafeName}-black-white.png`">Download</a></div>
        <div class="image-frame white-frame"><img :src="blackWhite" alt="Black QR code on white background" /></div>
      </article>
      <article class="output-card">
        <div class="card-heading"><div><p class="format">PNG · transparent</p><h2>White on transparent</h2></div><div class="card-actions"><button class="background-button" type="button" @click="isDarkPreview = !isDarkPreview">{{ isDarkPreview ? 'Light background' : 'Black background' }}</button><a :href="whiteTransparent" :download="`${fileSafeName}-white-transparent.png`">Download</a></div></div>
        <div class="image-frame transparent-frame" :class="{ 'dark-preview': isDarkPreview }"><img :src="whiteTransparent" alt="White QR code on transparent background" /></div>
      </article>
    </section>
  </main>
</template>
