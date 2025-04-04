<script setup>
import MonthCalendar from './Month.vue'

// Define props for the year
defineProps({
  year: {
    type: Number,
    required: true,
  },
})

// Array of month numbers (1 to 12)
const months = Array.from({ length: 12 }, (_, i) => i + 1)
</script>

<template>
  <header>
    <h1>{{ year }}</h1>
  </header>
  <div style="width: 100%">
    <div class="year-calendar">
      <div v-for="month in months" :key="month" class="months-container">
        <MonthCalendar :year="year" :month="month" />
      </div>
    </div>
  </div>
</template>

<style scoped>
h1 {
  text-align: center;
  margin: 20px 0;
}
.year-calendar {
  display: grid;
  grid-template-columns: repeat(4, 1fr); /* 4 columns */
  gap: 20px; /* Space between months */
  padding: 10px;
}

.months-container {
  border: 1px solid #ccc;
  border-radius: 5px;
  width: 100%;
}
</style>
