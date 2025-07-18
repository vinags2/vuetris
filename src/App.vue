<script setup>
import { ref } from "vue";
import TheGame from "./components/TheGame.vue";
import Welcome from "./components/Welcome.vue";

const showWelcome = ref(
  (localStorage.getItem("showWelcome") ?? "true") == "true",
);

const isLocalStorageEnabled = () => {
  try {
    const key = `__storage__test`;
    localStorage.setItem(key, null);
    localStorage.removeItem(key);
    return true;
  } catch (e) {
    return false;
  }
};

const localStorageAvailable = isLocalStorageEnabled();
</script>

<template>
  <div v-if="showWelcome">
    <Welcome
      @show-welcome="showWelcome = false"
      :local-storage-available="localStorageAvailable"
    ></Welcome>
  </div>
  <div v-else>
    <TheGame :local-storage-available="localStorageAvailable"></TheGame>
  </div>
</template>
