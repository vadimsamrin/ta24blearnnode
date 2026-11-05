<script setup>
import { onBeforeUnmount, ref } from 'vue'

const scrollArea = ref(null)
const isObserving = ref(false)
const error = ref('')
const result = ref('Click "Start observing", then scroll through the cards.')
const cardStates = ref({})

const cards = [
  { id: 1, title: 'Card 1', text: 'The observer checks whether this card is visible.' },
  { id: 2, title: 'Card 2', text: 'Scroll the container to change its visibility.' },
  { id: 3, title: 'Card 3', text: 'The visibility threshold is 50 percent.' },
  { id: 4, title: 'Card 4', text: 'Observer events update the result on this page.' },
  { id: 5, title: 'Card 5', text: 'Intersection Observer reports changes asynchronously.' },
  { id: 6, title: 'Card 6', text: 'Scroll to the bottom, then return to the top.' },
]

let observer = null

function startObserving() {
  error.value = ''

  if (!('IntersectionObserver' in window)) {
    error.value = 'Intersection Observer API is not supported in this browser.'
    result.value = 'Observation was not started.'
    return
  }

  if (!scrollArea.value) {
    error.value = 'The scroll area could not be found.'
    return
  }

  stopObserving(false)
  cardStates.value = {}

  try {
    observer = new IntersectionObserver((entries) => {
      for (const entry of entries) {
        const id = Number(entry.target.dataset.cardId)
        const ratio = Math.round(entry.intersectionRatio * 100)
        const visible = entry.isIntersecting && ratio >= 50

        cardStates.value = {
          ...cardStates.value,
          [id]: { visible, ratio },
        }

        result.value = `${entry.target.dataset.cardTitle}: ${visible ? 'visible' : 'not visible'} (${ratio}% of the card is in the scroll area).`
      }
    }, {
      root: scrollArea.value,
      threshold: [0, 0.5, 1],
    })

    scrollArea.value.querySelectorAll('[data-card-id]').forEach((card) => {
      observer.observe(card)
    })

    isObserving.value = true
    result.value = 'Intersection Observer is running. Scroll through the cards.'
  } catch (cause) {
    observer?.disconnect()
    observer = null
    isObserving.value = false
    error.value = `Could not start observation: ${cause.message}`
  }
}

function stopObserving(showResult = true) {
  observer?.disconnect()
  observer = null
  isObserving.value = false

  if (showResult) {
    result.value = 'Observation stopped.'
  }
}

onBeforeUnmount(() => stopObserving(false))
</script>

<template>
  <main class="page">
    <h1>My Web API Demo — Intersection Observer API</h1>
    <p>
      The Intersection Observer API detects when an element enters or leaves a scroll area.
      Start observing, then scroll through the cards to see the results.
    </p>

    <div ref="scrollArea" class="scroll-area" aria-label="Scrollable demo cards">
      <article
        v-for="card in cards"
        :key="card.id"
        class="demo-card"
        :class="{ 'is-visible': cardStates[card.id]?.visible }"
        :data-card-id="card.id"
        :data-card-title="card.title"
      >
        <div>
          <h2>{{ card.title }}</h2>
          <p>{{ card.text }}</p>
        </div>
        <span class="visibility-badge">
          {{ cardStates[card.id] ? `${cardStates[card.id].ratio}% · ${cardStates[card.id].visible ? 'visible' : 'not visible'}` : 'waiting' }}
        </span>
      </article>
    </div>

    <div class="buttons">
      <button :disabled="isObserving" @click="startObserving">Start observing</button>
      <button :disabled="!isObserving" class="secondary" @click="stopObserving()">Stop observing</button>
    </div>

    <p class="status" role="status">
      Status: <strong>{{ isObserving ? 'Observer is active' : 'Observer is stopped' }}</strong>
    </p>

    <section class="output" aria-live="polite">
      <h2>Result</h2>
      <p>{{ result }}</p>
    </section>

    <p v-if="error" class="error" role="alert">⚠️ {{ error }}</p>
  </main>
</template>

<style scoped>
.page {
  max-width: 800px;
  margin: 0 auto;
  padding: 24px;
  font-family: Arial, sans-serif;
}

.scroll-area {
  display: grid;
  gap: 14px;
  height: 340px;
  margin-top: 20px;
  overflow-y: auto;
  padding: 14px;
  border: 2px solid #b7c5d8;
  border-radius: 12px;
  background: #f3f6fb;
  scroll-behavior: smooth;
}

.demo-card {
  display: flex;
  min-height: 125px;
  align-items: center;
  justify-content: space-between;
  gap: 16px;
  padding: 18px;
  border: 2px solid #cbd5e1;
  border-radius: 10px;
  background: white;
  color: #1f2937;
  transition: border-color 180ms ease, background-color 180ms ease;
}

.demo-card h2,
.output h2 {
  margin: 0 0 8px;
  font-size: 1.1rem;
}

.demo-card p,
.output p {
  margin: 0;
}

.demo-card.is-visible {
  border-color: #16803c;
  background: #eaf8ef;
}

.visibility-badge {
  flex: 0 0 auto;
  border-radius: 999px;
  padding: 6px 10px;
  background: #e8edf5;
  color: #243247;
  font-size: 0.85rem;
}

.buttons {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
  margin-top: 18px;
}

button {
  padding: 10px 16px;
  border: 0;
  border-radius: 8px;
  background: #2563eb;
  color: white;
  cursor: pointer;
  font-size: 16px;
}

button.secondary {
  background: #475569;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.55;
}

.status {
  margin-top: 18px;
}

.output {
  margin-top: 20px;
  padding: 16px;
  border-radius: 8px;
  background: #f3f4f6;
  color: #111827;
}

.error {
  margin-top: 16px;
  padding: 12px;
  border-radius: 8px;
  background: #fee2e2;
  color: #991b1b;
}

@media (max-width: 560px) {
  .demo-card {
    align-items: flex-start;
    flex-direction: column;
  }
}
</style>
