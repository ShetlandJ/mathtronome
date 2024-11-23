<script setup>
import { ref, computed, onBeforeUnmount, watch } from "vue";

const props = defineProps({
  bars: {
    type: Array,
    default: () => [],
  },
  bpm: {
    type: Number,
    default: 120,
  },
});

const currentBar = ref(0);
const currentBeat = ref(0);
let interval = null;

// Method to calculate the duration of each beat based on BPM and time signature
const calculateBeatInterval = (bar) => {
  if (!bar) return 500; // Default to 500ms if no bar is found

  const noteDuration = (60000 / props.bpm) * 4; // Duration of a whole note in ms
  const beatDuration = noteDuration / bar.denominator; // Adjust for the time signature denominator
  console.log(beatDuration, bar.denominator);
  return beatDuration;
};

// Method to calculate the total duration of the current bar dynamically
const calculateBarDuration = (bar) => {
  if (!bar) return 2000; // Default to 2000ms for 4/4

  const beatDuration = calculateBeatInterval(bar); // Get beat duration for this specific bar
  return beatDuration * bar.numerator; // Multiply beat duration by number of beats in the bar
};

const beatInterval = ref(calculateBeatInterval(props.bars[currentBar.value]));

// Start the bar display (play the bars)
const startBarDisplay = () => {
  stopBarDisplay();
  currentBar.value = 0;
  currentBeat.value = 0;

  // Update the interval using the calculated beat duration
  interval = setInterval(() => {
    console.log(beatInterval.value);
    const currentBarData = props.bars[currentBar.value];

    // Move to the next beat, or the next bar if all beats are done
    if (currentBeat.value < currentBarData.numerator - 1) {
      currentBeat.value++;
    } else {
      currentBeat.value = 0;
      if (currentBar.value < props.bars.length - 1) {
        currentBar.value++;
      } else {
        currentBar.value = 0;
        clearInterval(interval); // Stop the interval when the last bar is reached
      }
    }
  }, beatInterval.value);
};

watch(
  () => props.bars[currentBar.value],
  (newBar) => {
    beatInterval.value = calculateBeatInterval(newBar);
  }
);

// Stop the bar display
const stopBarDisplay = () => {
  if (interval) {
    clearInterval(interval);
    interval = null;
  }
  currentBar.value = 0;
  currentBeat.value = 0;
};

onBeforeUnmount(() => {
  stopBarDisplay();
});
</script>

<template>
  <div>
    <div class="controls mb-4" v-if="bars.length > 0">
      <q-btn color="primary" @click="startBarDisplay">Play All</q-btn>
      <q-btn color="secondary" @click="stopBarDisplay">Stop</q-btn>
    </div>

    <!-- Bar display -->
    <div class="bar-display" v-for="(bar, barIndex) in bars" :key="barIndex">
      <div class="bar-container">
        <div
          v-for="(beat, beatIndex) in Array.from({ length: bar.numerator })"
          :key="beatIndex"
          :class="[
            'beat',
            {
              active: barIndex === currentBar && beatIndex === currentBeat,
            },
          ]"
        ></div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.bar-display {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 1rem;
}

.bar-container {
  display: flex;
  gap: 0.5rem;
}

.beat {
  width: 20px;
  height: 20px;
  background: #ccc;
  border-radius: 50%;
  transition: transform 0.2s ease, background-color 0.2s ease;
}

.beat.active {
  background: #4caf50;
  transform: scale(1.5);
  animation: pulse 0.4s ease-in-out;
}

@keyframes pulse {
  0%,
  100% {
    transform: scale(1.5);
  }
  50% {
    transform: scale(2);
  }
}
</style>
