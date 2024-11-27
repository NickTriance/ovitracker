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
* {
  margin:0;
  padding: 0;
  box-sizing: border-box;
}

.banner {
  width: 100%;
  height: 35vh;
  background-image: url('@/assets/header.jpg');
  background-size: cover;
  background-position: center;
  background-repeat: no-repeat;
  display: flex;
  align-items: center;
  justify-content: center;
}

.tracker-section {
  background-color: black;
  color: white;
  padding: 2rem;
  min-height: 100vh;
}
</style>
