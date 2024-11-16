<script setup>
import player from './components/player.vue';

</script>

<template>
  <header>
    <div class="wrapper">
      <h1>Player</h1>
    </div>
  </header>

  <main>
    <player/>
  </main>
</template>

<style scoped>
header {
  line-height: 1.5;
  display: flex;
  justify-content: center;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
