<script setup>
import { ref, shallowRef, computed, watch, nextTick, onMounted } from "vue";
import Chart from "chart.js/auto";

const weights = ref(JSON.parse(localStorage.getItem("weights") || "[]"));
const weightChartEl = ref(null);
const weightChart = shallowRef(null);
const weightInput = ref(60);

const currentweight = computed(() => {
  return weights.value.sort((a, b) => b.date - a.date)[0] || { weight: 0 };
});

const addWeight = () => {
  if (weightInput.value != "") {
    weights.value.push({
      weight: weightInput.value,
      date: new Date().getTime(),
    });
  }
};

const resetWeightsValue = () => {
  weights.value = [];
  localStorage.setItem("weights", JSON.stringify(weights));
};

const initializeChart = (newWeights) => {
  const ws = [...newWeights];

  if (weightChart.value) {
    weightChart.value.data.labels = ws
      .sort((a, b) => a.date - b.date)
      .map((weight) => new Date(weight.date).toLocaleDateString());

    weightChart.value.data.datasets[0].data = ws
      .sort((a, b) => a.date - b.date)
      .map((weight) => weight.weight);

    weightChart.value.update();
    return;
  }

  nextTick(() => {
    weightChart.value = new Chart(weightChartEl.value.getContext("2d"), {
      type: "line",
      data: {
        labels: ws
          .sort((a, b) => a.date - b.date)
          .map((weight) => new Date(weight.date).toLocaleDateString()),
        datasets: [
          {
            label: "Weight",
            data: ws
              .sort((a, b) => a.date - b.date)
              .map((weight) => weight.weight),
            backgroundColor: "rgba(0, 255, 0, 0.2)",
            borderColor: "rgba(0, 255, 0, 1)",
            borderWidth: 1,
            fill: true,
          },
        ],
      },
      options: {
        responsive: true,
        maintainAspectRatio: false,
      },
    });
  });
};

watch(
  weights,
  (newWeights) => {
    localStorage.setItem("weights", JSON.stringify(newWeights));
    initializeChart(newWeights);
  },
  { deep: true }
);

onMounted(() => {
  if (weights.value.length > 0) {
    initializeChart(weights.value);
  }
});
</script>

<template>
  <main class="flex flex-col min-h-screen font-Poppins bg-[#eee] text-center">
    <div class="container my-12">
      <h1 class="text-4xl text-center font-bold mb-8">Weight Tracker</h1>

      <div
        class="flex flex-col justify-center items-center w-52 h-52 text-center bg-white rounded-full shadow-md border-4 border-green-500 mx-auto mb-8"
      >
        <span class="block text-2xl font-bold mb-2">{{
          currentweight.weight
        }}</span>
        <small class="text-[#888] italic">Current weight (kg)</small>
      </div>

      <form
        @submit.prevent="addWeight"
        class="flex mb-12 rounded-md overflow-hidden transition-all duration-200 border-white-700"
      >
        <input
          type="number"
          step="0.1"
          v-model="weightInput"
          class="mr-3 appearance-none focus:outline-none border-none bg-white flex-grow p-4 text-xl"
        />
        <input
          type="submit"
          value="Add Weight"
          class="appearance-none rounded-md focus:outline-none border-none cursor-pointer bg-green-700 px-4 py-2 text-white text-xl font-bold transition-all duration-200 border-4 border-transparent hover:bg-white hover:text-green-700 hover:border-4 hover:border-green-700"
        />
      </form>

      <div v-show="weights.length">
        <div class="flex justify-between mb-6">
          <h2 class="mb-4 text-gray-500 font-normal">Last 7 days</h2>
          <button
            class="focus:outline-none border-0 cursor-pointer bg-green-700 px-4 py-2 text-white text-base font-bold rounded-md transition-all duration-200 border-l-4 border-transparent"
            @click="resetWeightsValue"
          >
            Reset
          </button>
        </div>

        <div class="w-full bg-white p-4 rounded-md shadow-md mb-8">
          <canvas ref="weightChartEl"></canvas>
        </div>

        <div class="weight-history">
          <h2 class="mb-4 text-gray-500 font-normal text-left">
            Weight History
          </h2>
          <ul class="list-none m-0 p-0">
            <li class="list-item" v-for="weight in weights">
              <span>{{ weight.weight }}kg</span>
              <small>{{ new Date(weight.date).toLocaleDateString() }}</small>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
/* This is where you combine traditional CSS with Tailwind classes */
.list-item {
  @apply flex justify-between items-center p-2 cursor-pointer;
}

.list-item:nth-child(even) {
  @apply bg-gray-300;
}

.list-item:hover {
  @apply hover:bg-gray-200;
}

.list-item:last-of-type {
  @apply border-b-0;
}
</style>
