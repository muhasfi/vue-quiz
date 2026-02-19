<script setup>
const { quizQuestionsLength, numberOfCorrectAnswer } = defineProps([
  "quizQuestionsLength",
  "numberOfCorrectAnswer",
]);
</script>

<template>
  <section class="results">
    <p>Score:</p>
    <h1>{{ numberOfCorrectAnswer }} / {{ quizQuestionsLength }}</h1>
    <RouterLink to="/">Go Back</RouterLink>
  </section>
</template>
