<script setup>
import { ref, onMounted, onBeforeUnmount } from 'vue'

const demoBox = ref(null)
const result = ref('Fullscreen mode is not active yet.')
const error = ref(null)
const isFullscreen = ref(false)

function updateStatus() {
  const activeElement = document.fullscreenElement

  isFullscreen.value = activeElement === demoBox.value

  if (activeElement) {
    result.value = `Fullscreen is active.
Current fullscreen element: ${activeElement.tagName.toLowerCase()}`
  } else {
    result.value = 'Fullscreen mode is not active.'
  }
}

async function runDemo() {
  error.value = null

  if (!document.fullscreenEnabled) {
    error.value = 'Fullscreen API is not supported or is blocked in this browser.'
    return
  }

  if (!demoBox.value) {
    error.value = 'Demo element was not found.'
    return
  }

  try {
    await demoBox.value.requestFullscreen()
    result.value = 'Fullscreen request was successful.'
  } catch (err) {
    error.value = err.message || 'The browser denied the fullscreen request.'
  }
}

async function exitDemo() {
  error.value = null

  if (!document.fullscreenElement) {
    result.value = 'Fullscreen is already turned off.'
    return
  }

  try {
    await document.exitFullscreen()
    result.value = 'Exited fullscreen mode.'
  } catch (err) {
    error.value = err.message || 'Could not exit fullscreen mode.'
  }
}

function handleFullscreenError() {
  error.value = 'Fullscreen error happened. The browser did not allow fullscreen mode.'
}

onMounted(() => {
  document.addEventListener('fullscreenchange', updateStatus)
  document.addEventListener('fullscreenerror', handleFullscreenError)
})

onBeforeUnmount(() => {
  document.removeEventListener('fullscreenchange', updateStatus)
  document.removeEventListener('fullscreenerror', handleFullscreenError)
})
</script>

<template>
  <div class="page">
    <h1>My Web API Demo — Fullscreen API</h1>

    <p>
      This page demonstrates the Fullscreen API. It can open one specific
      element in fullscreen mode and then exit fullscreen mode.
    </p>

    <div ref="demoBox" class="demo-box">
      <h2>Fullscreen Demo Box</h2>

      <p>
        Click the button below to make this box fullscreen.
      </p>

      <p>
        Status:
        <strong>{{ isFullscreen ? 'Fullscreen is active' : 'Fullscreen is not active' }}</strong>
      </p>

      <div class="buttons">
        <button @click="runDemo">
          Enter Fullscreen
        </button>

        <button @click="exitDemo">
          Exit Fullscreen
        </button>
      </div>
    </div>

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
  padding: 24px;
  border: 2px solid #333;
  border-radius: 12px;
  background: white;
  color: black;
}

.demo-box:fullscreen {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;

  background: #202124;
  color: white;
  font-size: 24px;
}

.buttons {
  display: flex;
  gap: 12px;
  margin-top: 16px;
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