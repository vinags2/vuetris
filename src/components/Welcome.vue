<script setup>
import { ref } from "vue";

const props = defineProps({
  localStorageAvailable: Boolean,
});

const emit = defineEmits(["showWelcome"]);

const name = ref(
  props.localStorageAvailable
    ? localStorage.getItem("playerName") || ""
    : "anonymous",
);

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
          type="text"
          v-model="name"
          autofocus
          class="mt-1 block w-full rounded-md border-gray-300 text-black shadow-sm focus:border-emerald-500 focus:ring-emerald-500 sm:text-sm"
        />
      </div>
      <div class="mb-4 flex items-center space-x-2">
        <input
          id="show-welcome"
          type="checkbox"
          v-model="showWelcome"
          autofocus
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
