<script setup>
import { ref } from "vue";
const emit = defineEmits(["close"]);

const showWelcome = ref(localStorage.getItem("showWelcome") === "true");

function saveSettings() {
  localStorage.setItem("showWelcome", showWelcome.value);
  emit("close");
}
</script>
<template>
  <div
    class="flex h-screen flex-col items-center justify-center bg-emerald-900"
  >
    <div
      class="border-b-2 border-gray-200 bg-emerald-700 px-4 py-8 text-center text-emerald-100"
    >
      <h1 class="mb-4 text-4xl font-bold">Settings</h1>
      <div class="mb-4 flex items-center space-x-2">
        <input
          id="show-welcome"
          type="checkbox"
          v-model="showWelcome"
          autofocus
          class="rounded-md border-gray-300 text-black shadow-sm focus:border-emerald-500 focus:ring-emerald-500 sm:text-sm"
        />
        <label for="show-welcome" class="text-sm font-medium text-gray-200">
          Show Welcome Screen on Start
        </label>
      </div>
      <div class="flex items-center">
        <button
          class="rounded bg-emerald-600 px-6 py-2 font-bold transition-colors hover:bg-emerald-500"
          @click="$emit('close')"
        >
          Cancel
        </button>
        <button
          class="ml-4 rounded bg-emerald-600 px-6 py-2 font-bold transition-colors hover:bg-emerald-500"
          @click="saveSettings"
        >
          Save
        </button>
      </div>
    </div>
  </div>
</template>
