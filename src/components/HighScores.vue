<script setup>
import { onMounted, ref, watch } from "vue";

const props = defineProps({
  name: String,
  score: Number | String,
  saveScore: Boolean,
});

const emit = defineEmits(["scoreSaved"]);

const scores = ref([]);

const saveScores = () => {
  scores.value = JSON.parse(localStorage.getItem("scores")) ?? [];
  scores.value?.push({
    key: scores.value.length,
    name: props.name,
    value: props.score,
  });

  localStorage.setItem(
    "scores",
    JSON.stringify(
      scores.value?.sort((a, b) => b.value - a.value).slice(0, 10),
    ),
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

onMounted(() => {
  showScores();
});

watch(
  () => props.saveScore,
  (newValue) => {
    if (newValue) {
      saveScores();
    }
  },
);
</script>
<template>
  <div class="mb-6">
    <h2 class="mb-2 text-xl font-bold">High Scores:</h2>
  </div>
  <div>
    <ul class="list-disc pl-6">
      <li v-for="(score, index) in scores" :key="index" class="mb-1">
        <!-- {{ score.name }} : {{ score.value }} -->
        {{ score.value }}
      </li>
    </ul>
  </div>
  <div>
    <button
      class="mt-6 rounded bg-emerald-600 px-6 py-2 font-bold transition-colors hover:bg-emerald-500"
      @click="clearScores"
    >
      Clear Scores
    </button>
    <button
      v-if="props.saveScore"
      class="mt-6 rounded bg-emerald-600 px-6 py-2 font-bold transition-colors hover:bg-emerald-500"
      @click="saveScores"
    >
      Save Scores
    </button>
  </div>
</template>
