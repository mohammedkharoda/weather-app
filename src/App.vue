<script setup>
import { RouterView } from "vue-router";
import SiteNavigation from "../src/components/SiteNavigation.vue";
</script>

<template>
  <div class="flex flex-col min-h-screen font-Roboto bg-weather-primary">
    <SiteNavigation />
    <RouterView />
  </div>
</template>

<style lang="scss" scoped></style>
