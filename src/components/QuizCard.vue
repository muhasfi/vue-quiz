<script setup>
import { defineProps } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();
const { quiz } = defineProps(["quiz"]);
// console.info(quiz);

function goToQuiz() {
  router.push({ name: "quiz", params: { id: quiz.id } });
}
</script>

<template>
  <div class="card" @click="goToQuiz">
    <img :src="quiz.img" :alt="quiz.title" />
    <div class="card-body">
      <h2>{{ quiz.title }}</h2>
      <p>{{ quiz.questions.length }} Questions</p>
    </div>
  </div>
</template>

<style scoped>
.card img {
  width: 100%;
  height: 200px;
  object-fit: cover;
  margin: 0;
}
.card-body {
  padding: 0 15px;
}

.card-body h2 {
  font-weight: bold;
}
</style>
