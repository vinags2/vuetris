<script setup>
import { onMounted, ref, watch, computed } from "vue";
const props = defineProps({
  score: Number,
  saveScore: Boolean,
  newGameStarted: Boolean,
});

const emit = defineEmits(["scoreSaved"]);

const scores = ref([]);
const playerName = ref(localStorage.getItem("playerName") || "anonymous");
const isNewHighScore = ref(false);
const maxScores = 12;

const saveScores = () => {
  scores.value?.push({
    key: scores.value.length,
    name: playerName.value,
    value: props.score,
  });
  scores.value.slice(0, maxScores - 1);

  localStorage.setItem(
    "scores",
    JSON.stringify(scores.value?.sort((a, b) => b.value - a.value)),
  );
  emit("scoreSaved");
};

const showScores = () => {
  scores.value = JSON.parse(localStorage.getItem("scores"));
};

const clearScores = () => {
  if (window.confirm("Are you sure you want to clear the scores?")) {
    localStorage.removeItem("scores");
    scores.value = [];
  }
};

// Helper function to display player name
// Truncate name to 15 characters for display
function displayName(name) {
  return (name ?? "anonymous").substring(0, 15);
}

onMounted(() => {
  showScores();
});

const newHighScore = () => {
  if (props.score === 0) {
    isNewHighScore.value = false;
  } else if (scores.value.length < maxScores) {
    isNewHighScore.value = true;
  } else {
    isNewHighScore.value = scores.value.some(
      (score) => score.value < props.score,
    );
  }

  return isNewHighScore.value;
};

watch(
  () => props.newGameStarted,
  (newValue) => {
    if (newValue) {
      isNewHighScore.value = false;
    }
  },
);

watch(
  () => props.saveScore,
  (newValue) => {
    if (newValue && newHighScore()) {
      saveScores();
    }
  },
);
</script>
<template>
  <div v-if="isNewHighScore" class="mb-6">
    <h2 class="mb-2 animate-pulse text-xl font-bold text-red-500">
      A New High Score!
    </h2>
  </div>
  <div v-else class="mb-6">
    <h2 class="mb-2 text-xl font-bold">High Scores:</h2>
  </div>
  <div>
    <ul class="">
      <li v-for="(score, index) in scores" :key="index" class="mb-1">
        <div
          :class="{
            'grid grid-cols-3 gap-1': true,
            'text-red-500':
              props.score === score.value && playerName == score.name,
          }"
        >
          <div>{{ displayName(score.name) }}</div>
          <div class="col-span-2 justify-self-end text-right">
            {{ score.value.toLocaleString() }}
          </div>
        </div>
      </li>
    </ul>
  </div>
  <div class="space-y-2 pt-4">
    <button
      class="w-full rounded bg-emerald-600 py-2 font-bold transition-colors hover:bg-emerald-500"
      @click="clearScores"
    >
      Clear Scores
    </button>
  </div>
</template>
