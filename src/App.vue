<script setup>
import { ref, shallowRef, computed, watch, nextTick } from "vue";
import Chart from "chart.js/auto";

const widths = ref([]);
const widthChartEl = ref(null);
const widthChart = shallowRef(null);
const widthInput = ref(60.0);
const currentWidth = computed(() => {
  return widths.value.sort((a, b) => b.date - a.date[0] || { width: 0 });
});
const addWidth = () => {
  widths.value.push({
    width: widthInput.value,
    date: new Date().getTime(),
  });
};
</script>

<template>
  <div>
    <h1>Hello World</h1>
  </div>
</template>

<style scoped></style>
