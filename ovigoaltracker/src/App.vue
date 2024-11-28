<script setup lang="ts">
//@ts-ignore
import Tracker from './components/Tracker.vue'
</script>

<template>
  <div>
    <header class="banner">
    </header>

    <section class="tracker-section">
      <div class="wrapper">
        <Tracker />
      </div>
    </section>
  </div>
</template>

<style scoped>

.banner {
  width: 100%;
  height: 35vh;
  background-image: url('@/assets/header.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  flex-shrink: 0;
}

.tracker-section {
  background-color: black;
  color: white;
  padding: 2rem;
  height: calc(100vh - 35vh);
  width: 100%;
  overflow: hidden;
}
</style>
