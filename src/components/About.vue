<script setup>
import GitHubIcon from "./GitHubIcon.vue";
const emit = defineEmits(["close"]);
</script>
<template>
  <div
    class="flex h-screen flex-col items-center justify-center bg-emerald-900"
  >
    <div
      class="w-250 max-w-lg justify-between rounded-lg border-b-2 border-gray-200 bg-emerald-700 px-4 py-8 text-emerald-100 shadow-lg"
    >
      <div class="flex items-center justify-center">
        <h1 class="mb-4 text-4xl font-bold">About Vuetris</h1>
      </div>
      <div>
        'vuetris' is a game based on the classic Tetris game, and is intended
        for personal use, and programming instruction.
      </div>
      <div class="mt-4">
        The development of the application, the logic and the appearance was
        started in June, 2025 by
        <a
          class="cursor-pointer text-emerald-300 hover:text-emerald-200"
          href="https://github.com/Mint-Diary/vuetris"
          target="_blank"
          rel="noopener noreferrer"
        >
          Demid Krom (aka Mint-Diary)
        </a>
      </div>
      <div class="mt-4">
        Further development, including a High Scores table, started in July,
        2025 by
        <a
          class="cursor-pointer text-emerald-300 hover:text-emerald-200"
          target="_blank"
          href="https://gregvinall.com"
        >
          Greg Vinall</a
        >
      </div>
      <div class="mt-4">
        The
        <a
          class="cursor-pointer text-emerald-300 hover:text-emerald-200"
          target="_blank"
          href="https://github.com/vinags2/vuetris"
          >source code</a
        >
        is available on GitHub and is licensed under the
        <a
          class="cursor-pointer text-emerald-300 hover:text-emerald-200"
          target="_blank"
          href="https://mit-license.org"
          >MIT licence</a
        >
        and licenses imposed by Tetris.com
      </div>
      <div class="mt-8 flex items-center justify-center space-x-2">
        <button
          class="rounded bg-emerald-600 px-6 py-2 font-bold transition-colors hover:bg-emerald-500"
          @click="$emit('close')"
        >
          Close
        </button>
      </div>
    </div>
  </div>
</template>
