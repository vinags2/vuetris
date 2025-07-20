<script setup>
import { computed, ref, onMounted } from "vue";

const props = defineProps({
  localStorageAvailable: Boolean,
});

const emit = defineEmits(["showWelcome"]);

const name = ref("anonymous");

const today = computed(() => {
  return new Date().toLocaleDateString("en-AU", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
  });
});

const useDateInHighScores = ref(
  props.localStorageAvailable
    ? (localStorage.getItem("useDate") || "false") == "true"
    : false,
);

function setName() {
  if (useDateInHighScores.value) {
    return today.value;
  } else if (props.localStorageAvailable) {
    return localStorage.getItem("playerName");
  }
}

let oldName = name.value;
const nameRef = ref(null);

function useDate(e) {
  if (!e.target.checked) {
    name.value = oldName;
  } else {
    oldName = name.value;
    name.value = today.value;
  }
  if (props.localStorageAvailable)
    localStorage.setItem("useDate", !useDateInHighScores.value);
  nameRef.value.focus();
}

function startGame() {
  saveSettings();
  if (props.localStorageAvailable)
    localStorage.setItem("playerName", name.value);
  emit("showWelcome", false);
}
const showWelcome = ref(
  props.localStorageAvailable
    ? (localStorage.getItem("showWelcome") ?? "true" === "true")
    : true,
);

function saveSettings() {
  if (props.localStorageAvailable)
    localStorage.setItem("showWelcome", showWelcome.value);
}

onMounted(() => {
  name.value = setName();
});
</script>

<template>
  <div
    class="flex h-screen flex-col items-center justify-center bg-emerald-900"
  >
    <div
      class="border-b-2 border-gray-200 bg-emerald-700 px-4 py-8 text-center text-emerald-100"
    >
      <h1 class="mb-4 text-4xl font-bold">Vuetris</h1>
      <p class="mb-4 text-lg">Welcome to vuetris, a tetris clone</p>
      <div class="mb-4">
        <label for="name" class="block text-sm font-medium text-gray-200">
          Enter a short name to be used for your high scores:
        </label>
        <input
          id="name"
          ref="nameRef"
          type="text"
          v-model="name"
          autofocus
          class="mt-1 block w-full rounded-md border-gray-300 text-black shadow-sm focus:border-emerald-500 focus:ring-emerald-500 sm:text-sm"
        />
      </div>
      <div class="mb-4 flex items-center space-x-2">
        <input
          @click="useDate($event)"
          v-model="useDateInHighScores"
          id="use-date"
          type="checkbox"
          class="rounded-md border-gray-300 text-black shadow-sm focus:border-emerald-500 focus:ring-emerald-500 sm:text-sm"
        />
        <label for="use-date" class="text-sm font-medium text-gray-200">
          Use date in high scores
        </label>
      </div>
      <div class="mb-4 flex items-center space-x-2">
        <input
          id="show-welcome"
          type="checkbox"
          v-model="showWelcome"
          class="rounded-md border-gray-300 text-black shadow-sm focus:border-emerald-500 focus:ring-emerald-500 sm:text-sm"
        />
        <label for="show-welcome" class="text-sm font-medium text-gray-200">
          Show this screen on start
        </label>
      </div>
      <button
        class="mt-6 rounded bg-emerald-600 px-6 py-2 font-bold transition-colors hover:bg-emerald-500"
        @click="startGame"
      >
        Start Game
      </button>
    </div>
  </div>
</template>
