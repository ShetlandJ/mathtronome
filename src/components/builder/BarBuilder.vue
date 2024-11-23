<script setup>
import { ref } from "vue";
import BarDisplay from "./BarDisplay.vue";

const bpm = ref(120);

const numerator = ref(4);
const denominator = ref(4);
const availableDenominators = [2, 4, 8, 16, 32];

const bars = ref([
    {
        order: 1,
        numerator: 4,
        denominator: 4,
    },
    {
        order: 2, 
        numerator: 7,
        denominator: 8,
    }
]);

const currentBar = ref(null);
const isPlaying = ref(false);

const addBar = () => {
  bars.value.push({
    order: bars.value.length + 1,
    numerator: numerator.value,
    denominator: denominator.value,
  });
};

const playAllBars = () => {
  if (!bars.value.length) return;
  currentBar.value = 1;
  isPlaying.value = true;
};

const stop = () => {
  currentBar.value = null;
  isPlaying.value = false;
};

const onBarComplete = () => {
  if (!isPlaying.value) return;

  const nextBar = currentBar.value + 1;
  if (nextBar <= bars.value.length) {
    currentBar.value = nextBar;
  } else {
    stop();
  }
};
</script>

<template>
    <div>
      <header class="bg-white shadow">
        <div class="max-w-7xl mx-auto py-6 px-4 sm:px-6 lg:px-8">
          <h1 class="text-3xl font-bold text-gray-900">Bar Builder</h1>
        </div>
      </header>
  
      <div id="controls mb-4">
        <!-- BPM -->
        <div class="items-center justify-between p-4">
          <div class="mb-2">
            <label for="bpm" class="text-lg font-semibold text-gray-800 mr-2">
              BPM
            </label>
            <input
              type="number"
              v-model="bpm"
              id="bpm"
              class="w-16 h-10 text-lg text-center border border-gray-300 rounded-md"
            />
          </div>
  
          <!-- Time Signature -->
          <div>
            <label
              for="time-signature"
              class="text-lg font-semibold text-gray-800 mr-2"
            >
              Time Signature
            </label>
            <input
              type="number"
              v-model="numerator"
              id="numerator"
              class="w-16 h-10 text-lg text-center border border-gray-300 rounded-md"
            />
            <select
              v-model="denominator"
              id="denominator"
              class="w-16 h-10 text-lg text-center border border-gray-300 rounded-md"
            >
              <option
                v-for="denominator in availableDenominators"
                :key="denominator"
                :value="denominator"
              >
                {{ denominator }}
              </option>
            </select>
          </div>
        </div>
      </div>
  
      <div class="button-group mb-4">
        <q-btn color="primary" label="Add bar" @click="addBar" />
      </div>
    
      <!-- Render BarDisplay Components -->
      <BarDisplay
        :bars="bars"
      />
    </div>
  </template>