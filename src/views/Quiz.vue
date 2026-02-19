<script setup>
import QuizContent from "@/components/QuizContent.vue";
import QuizHeader from "@/components/QuizHeader.vue";
import QuizResult from "@/components/QuizResult.vue";
import { useRoute } from "vue-router";
import quizes from "../data/quiz.json";
import { computed, ref, watch } from "vue";
import QuizNavigation from "@/components/QuizNavigation.vue";

const route = useRoute();
const quizId = parseInt(route.params.id);
const quiz = quizes.find((q) => q.id === quizId);

const numberOfCorrectAnswer = ref(0);
const currentQuestionIndex = ref(0);
const showResult = ref(false);
const showConfirm = ref(false);

const questionPage = computed(() => {
  return `${currentQuestionIndex.value + 1} / ${quiz.questions.length}`;
});

const barPercentage = computed(() => {
  return `${((currentQuestionIndex.value + 1) / quiz.questions.length) * 100}%`;
});

function onSelectOption(option) {
  if (option.correct) {
    numberOfCorrectAnswer.value++;
  }
  if (currentQuestionIndex.value === quiz.questions.length - 1) {
    showResult.value = true;
    // showConfirm.value = true;
    return;
  }
  currentQuestionIndex.value++;
}
</script>

<template>
  <!-- <div v-if="showConfirm" class="confirm-box">
    <p>Yakin ingin menyelesaikan quiz?</p>
    <button @click="showResult = true">Ya</button>
    <button @click="showConfirm = false">Tidak</button>
  </div> -->

  <QuizHeader :questionPage="questionPage" :barPercentage="barPercentage" />
  <QuizContent
    v-if="!showResult"
    :question="quiz.questions[currentQuestionIndex]"
    @selectOption="onSelectOption"
  />
  <QuizResult
    v-else
    :quizQuestionsLength="quiz.questions.length"
    :numberOfCorrectAnswer="numberOfCorrectAnswer"
  />
  <QuizNavigation
    :currentQuestionIndex="currentQuestionIndex"
    :totalQuestions="quiz.questions.length"
    :showResult="showResult"
    @next="currentQuestionIndex++"
    @prev="currentQuestionIndex--"
  />
</template>

<style scoped></style>
