<script setup>
import { ref } from "vue";
import srcQuiz from "../data/quiz.json";
import QuizCard from "../components/QuizCard.vue";

const quizes = ref(srcQuiz);
</script>

<template>
  <header>
    <h1 id="title">Quiz Vue</h1>
    <input type="text" id="search-input" />
  </header>
  <section id="quiz-container">
    <QuizCard v-for="quiz in quizes" :key="quiz.id" :quiz="quiz" />
  </section>
</template>

<style scoped>
header {
  margin-top: 30px;
  margin-bottom: 10px;
  display: flex;
  align-items: center;
}

#title {
  font-weight: bold;
  margin-right: 30px;
}

#search-input {
  border: none;
  background-color: rgb(184, 184, 184);
  padding: 10px;
  border-radius: 10px;
}

#quiz-container {
  display: flex;
  flex-wrap: wrap;
  margin-top: 20px;
}

.card {
  width: 270px;
  margin-right: 30px;
  margin-bottom: 30px;
  border-radius: 5px;
  overflow: hidden;
  box-shadow: 0 0 5px rgba(0, 0, 0, 0.1);
}
</style>
