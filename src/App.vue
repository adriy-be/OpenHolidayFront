<script setup>
import Year from './components/Year.vue'
</script>

<template>
  <header></header>

  <main>
    <Year :year="2025" />
    <Year :year="2026" />
  </main>
</template>
<style scoped></style>
