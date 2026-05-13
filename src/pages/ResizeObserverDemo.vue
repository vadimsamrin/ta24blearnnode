<script setup>
import { ref, onBeforeUnmount } from 'vue'

const demoBox = ref(null)
const result = ref('Click "Start observing" to begin.')
const error = ref(null)
const observing = ref(false)

const boxWidth = ref(260)
const boxHeight = ref(120)

let observer = null

function startDemo() {
  error.value = null

  if (!('ResizeObserver' in window)) {
    error.value = 'Resize Observer API is not supported in this browser.'
    return
  }

  if (!demoBox.value) {
    error.value = 'Demo box was not found.'
    return
  }

  observer = new ResizeObserver((entries) => {
    const entry = entries[0]
    const width = Math.round(entry.contentRect.width)
    const height = Math.round(entry.contentRect.height)

    result.value = `The box size changed.
Width: ${width}px
Height: ${height}px`
  })

  observer.observe(demoBox.value)
  observing.value = true
  result.value = 'Resize Observer is now watching the box.'
}

function makeBigger() {
  boxWidth.value += 40
  boxHeight.value += 20
}

function makeSmaller() {
  if (boxWidth.value > 180) {
    boxWidth.value -= 40
  }

  if (boxHeight.value > 80) {
    boxHeight.value -= 20
  }
}

function stopDemo() {
  if (observer) {
    observer.disconnect()
    observer = null
  }

  observing.value = false
  result.value = 'Resize Observer stopped watching the box.'
}

onBeforeUnmount(() => {
  stopDemo()
})
</script>

<template>
  <div class="page">
    <h1>My Web API Demo — Resize Observer API</h1>

    <p>
      This page demonstrates the Resize Observer API.
      It watches one box and shows its current size when the box changes.
    </p>

    <div
      ref="demoBox"
      class="demo-box"
      :style="{ width: boxWidth + 'px', height: boxHeight + 'px' }"
    >
      <strong>Resizable box</strong>
      <span>{{ boxWidth }}px × {{ boxHeight }}px</span>
    </div>

    <div class="buttons">
      <button @click="startDemo">
        Start observing
      </button>

      <button @click="makeBigger">
        Make box bigger
      </button>

      <button @click="makeSmaller">
        Make box smaller
      </button>

      <button @click="stopDemo">
        Stop observing
      </button>
    </div>

    <p>
      Status:
      <strong>{{ observing ? 'Observer is active' : 'Observer is not active' }}</strong>
    </p>

    <div class="output">
      <h3>Result</h3>
      <pre>{{ result }}</pre>
    </div>

    <div v-if="error" class="error">
      ⚠️ {{ error }}
    </div>
  </div>
</template>

<style scoped>
.page {
  padding: 24px;
  max-width: 800px;
  margin: 0 auto;
  font-family: Arial, sans-serif;
}

.demo-box {
  margin-top: 20px;
  padding: 16px;
  border: 2px solid #333;
  border-radius: 12px;
  background: white;
  color: black;
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 8px;
  transition: width 0.3s, height 0.3s;
}

.buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 20px;
}

button {
  padding: 10px 16px;
  cursor: pointer;
  border: none;
  border-radius: 8px;
  background: #2563eb;
  color: white;
  font-size: 16px;
}

button:hover {
  background: #1d4ed8;
}

.output {
  margin-top: 24px;
  padding: 16px;
  background: #f3f4f6;
  color: black;
  border-radius: 8px;
}

pre {
  white-space: pre-wrap;
}

.error {
  margin-top: 16px;
  padding: 12px;
  background: #fee2e2;
  color: #991b1b;
  border-radius: 8px;
}
</style>