<script setup>
import { ref, onBeforeUnmount, watch } from "vue";

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

const currentBar = ref(null);
const currentBeat = ref(0);
let animationFrameId = null;

const calculateBeatInterval = (bar) => {
  if (!bar) return 500;

  const noteDuration = (60000 / props.bpm) * 4;
  const beatDuration = noteDuration / bar.denominator;
  return beatDuration;
};

const beatInterval = ref(calculateBeatInterval(props.bars[currentBar.value]));

let startTime = 0;
let totalElapsedTime = 0;

const startBarDisplay = () => {
  stopBarDisplay();
  currentBeat.value = 0;
  totalElapsedTime = 0;
  startTime = performance.now();

  if (currentBar.value === null) {
    currentBar.value = 0;
  }

  const updateLoop = () => {
    const currentBarData = props.bars[currentBar.value];
    const currentTime = performance.now();
    const elapsedTime = currentTime - startTime + totalElapsedTime;

    const beatsPerBar = currentBarData.numerator;
    const beatDuration = calculateBeatInterval(currentBarData);
    const totalBeatsDuration = beatsPerBar * beatDuration;

    if (elapsedTime >= totalBeatsDuration) {
      totalElapsedTime += totalBeatsDuration;
      currentBeat.value = 0;
      currentBar.value = (currentBar.value + 1) % props.bars.length;

      startTime = performance.now();
    } else {
      currentBeat.value = Math.floor(elapsedTime / beatDuration) % beatsPerBar;
    }

    animationFrameId = requestAnimationFrame(updateLoop);
  };

  animationFrameId = requestAnimationFrame(updateLoop);
};

const stopBarDisplay = () => {
  if (animationFrameId) {
    cancelAnimationFrame(animationFrameId);
    animationFrameId = null;
  }
  currentBeat.value = 0;
  totalElapsedTime = 0;
};

watch(beatInterval, () => {
  stopBarDisplay();
  startBarDisplay();
});

watch(
  () => props.bars[currentBar.value],
  (newBar) => {
    beatInterval.value = calculateBeatInterval(newBar);
  }
);

onBeforeUnmount(() => {
  stopBarDisplay();
});
</script>

<template>
  <div>
    <div class="controls mb-4 flex justify-center" v-if="bars.length > 0">
      <q-btn color="positive mr-2" @click="startBarDisplay">Play All</q-btn>
      <q-btn color="negative" @click="stopBarDisplay">Stop</q-btn>
    </div>

    <div class="bar-display" v-for="(bar, barIndex) in bars" :key="barIndex">
      <div class="bar-container">
        <p>{{ bar.numerator }} / {{ bar.denominator }}</p>
        <div
          v-for="(beat, beatIndex) in Array.from({ length: bar.numerator })"
          :key="beatIndex"
          class="mb-4"
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
