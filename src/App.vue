<template>
  <div class="min-h-screen w-full bg-white text-gray-800 flex flex-col">
    <router-view class="flex-1" />
    <PopupNotification />
  </div>
</template>

<script setup>
import PopupNotification from "./components/PopupNotification.vue";
</script>

<style scoped></style>
