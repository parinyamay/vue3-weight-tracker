<script setup lang="ts">
import { ref, shallowRef, computed, watch, nextTick } from 'vue'
import Chart from 'chart.js/auto'

interface Weight {
  weight: number
  date: number
}

const weights = ref<Weight[]>([])

const weightHistory = ref<Weight[]>([])

const weightChartEl = ref<any>(null)

const weightChart = shallowRef<any>(null)

const weightInput = ref<number>(60.0)

const currentWeight = computed(() => {
  return weights.value.slice().reverse()[0] || { weight: 0 }
})

watch(
  weights,
  (newWeights) => {
    const ws = [...newWeights]
    weightHistory.value = [...ws.reverse()]

    if (weightChart.value) {
      weightChart.value.data.labels = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => new Date(w.date).toLocaleDateString())
        .slice(-7)

      weightChart.value.data.datasets[0].data = ws
        .sort((a, b) => a.date - b.date)
        .map((w) => w.weight)
        .slice(-7)

      weightChart.value.update()

      return
    }

    nextTick(() => {
      weightChart.value = new Chart(weightChartEl.value.getContext('2d'), {
        type: 'line',
        data: {
          labels: ws
            .sort((a, b) => a.date - b.date)
            .map((w) => new Date(w.date).toLocaleDateString()),
          datasets: [
            {
              label: 'Weight',
              data: ws.sort((a, b) => a.date - b.date).map((w) => w.weight),
              backgroundColor: 'rgba(255,105,180, 0.2)',
              borderColor: 'rgba(255,105,180, 1)',
              fill: true
            }
          ]
        },
        options: {
          responsive: true,
          maintainAspectRatio: false
        }
      })
    })
  },
  { deep: true }
)

const addWeight = () => {
  weights.value.push({
    weight: weightInput.value,
    date: Date.now()
  })
}
</script>

<template>
  <main>
    <h1>Weight Tracker</h1>

    <div class="current">
      <span>{{ currentWeight?.weight }}</span>
      <small>Current weight (kg)</small>
    </div>

    <form @submit.prevent="addWeight">
      <input type="number" step="0.1" v-model="weightInput" />

      <input type="submit" value="Add weight" />
    </form>

    <div v-if="weights && weights.length > 0">
      <h2>Last 7 days</h2>

      <div class="canvas-box">
        <canvas ref="weightChartEl"></canvas>
      </div>

      <div class="weight-history">
        <h2>Weight History</h2>

        <ul>
          <li v-for="(weight, index) in weightHistory" :key="index">
            <span>{{ weight.weight }} kg</span>
            <small> {{ new Date(weight.date).toDateString() }}</small>
          </li>
        </ul>
      </div>
    </div>
  </main>
</template>
