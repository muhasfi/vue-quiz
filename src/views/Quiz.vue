<script setup>
import QuizContent from "@/components/QuizContent.vue";
import QuizHeader from "@/components/QuizHeader.vue";
import QuizResult from "@/components/QuizResult.vue";
import QuizNavigation from "@/components/QuizNavigation.vue";

import { useRoute } from "vue-router";
import quizes from "../data/quiz.json";
import { computed, ref } from "vue";

// ---------------------------------------------------------------------

const route = useRoute();
const quizId = parseInt(route.params.id);
const quiz = quizes.find((q) => q.id === quizId);

const currentQuestionIndex = ref(0);
const showResult = ref(false);
const showConfirm = ref(false);
const userAnswers = ref([]);

// ---------------------------------------------------------------------

// ✅ Hitung skor dengan membandingkan id jawaban
const numberOfCorrectAnswer = computed(() => {
  return quiz.questions.reduce((total, question, index) => {
    const correctAnswer = question.answers.find((a) => a.correct);
    if (userAnswers.value[index] === correctAnswer?.id) {
      return total + 1;
    }
    return total;
  }, 0);
});

// ✅ Progress text
const questionPage = computed(() => {
  return `${currentQuestionIndex.value + 1} / ${quiz.questions.length}`;
});

// ✅ Progress bar
const barPercentage = computed(() => {
  return `${Math.round(((currentQuestionIndex.value + 1) / quiz.questions.length) * 100)}%`;
});

// ✅ Cek apakah semua soal sudah dijawab
const allAnswered = computed(() => {
  return quiz.questions.every((_, index) => userAnswers.value[index] != null);
});

// ---------------------------------------------------------------------

function onSelectOption(option) {
  const index = currentQuestionIndex.value;
  userAnswers.value[index] = option.id;
}

function next() {
  if (currentQuestionIndex.value < quiz.questions.length - 1) {
    currentQuestionIndex.value++;
  }
}

function prev() {
  if (currentQuestionIndex.value > 0) {
    currentQuestionIndex.value--;
  }
}

function finishQuiz() {
  if (!allAnswered.value) {
    alert("Harap jawab semua soal sebelum menyelesaikan quiz.");
    return;
  }
  showConfirm.value = true;
}

function confirmFinish() {
  showResult.value = true;
  showConfirm.value = false;
}

function cancelFinish() {
  showConfirm.value = false;
}
</script>

<template>
  <div class="quiz-wrapper">
    <!-- Grid bg -->
    <div class="grid-bg"></div>
    <div class="orb orb-1"></div>
    <div class="orb orb-2"></div>

    <!-- Confirm Modal -->
    <Transition name="modal">
      <div v-if="showConfirm" class="confirm-overlay">
        <div class="confirm-box">
          <div class="confirm-icon">⚡</div>
          <h3 class="confirm-title">Selesaikan Quiz?</h3>
          <p class="confirm-desc">
            Pastikan semua jawaban sudah kamu isi dengan benar.
          </p>
          <div class="confirm-actions">
            <button class="btn-confirm-yes" @click="confirmFinish">
              Ya, Selesai!
            </button>
            <button class="btn-confirm-no" @click="cancelFinish">
              Cek Lagi
            </button>
          </div>
        </div>
      </div>
    </Transition>

    <div class="quiz-card">
      <!-- Back button -->
      <RouterLink to="/" class="back-button">
        <svg
          viewBox="0 0 24 24"
          fill="none"
          stroke="currentColor"
          stroke-width="2.5"
          width="14"
          height="14"
        >
          <path d="M19 12H5M12 5l-7 7 7 7" />
        </svg>
        Kembali
      </RouterLink>

      <QuizHeader :questionPage="questionPage" :barPercentage="barPercentage" />

      <QuizContent
        v-if="!showResult"
        :question="quiz.questions[currentQuestionIndex]"
        :selectedAnswer="userAnswers[currentQuestionIndex]"
        @selectOption="onSelectOption"
      />

      <QuizResult
        v-else
        :quizQuestionsLength="quiz.questions.length"
        :numberOfCorrectAnswer="numberOfCorrectAnswer"
        :questions="quiz.questions"
        :userAnswers="userAnswers"
      />

      <QuizNavigation
        v-if="!showResult && !showConfirm"
        :currentQuestionIndex="currentQuestionIndex"
        :totalQuestions="quiz.questions.length"
        :allAnswered="allAnswered"
        :questions="quiz.questions"
        :userAnswers="userAnswers"
        @next="next"
        @prev="prev"
        @finish="finishQuiz"
      />
    </div>
  </div>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Outfit:wght@400;500;600;700&display=swap");

* {
  box-sizing: border-box;
}

.quiz-wrapper {
  min-height: 100vh;
  background: #0a0a0f;
  display: flex;
  justify-content: center;
  align-items: center;
  padding: 24px;
  position: relative;
  overflow: hidden;
  font-family: "Outfit", sans-serif;
}

.grid-bg {
  position: fixed;
  inset: 0;
  background-image:
    linear-gradient(rgba(255, 165, 0, 0.025) 1px, transparent 1px),
    linear-gradient(90deg, rgba(255, 165, 0, 0.025) 1px, transparent 1px);
  background-size: 60px 60px;
  pointer-events: none;
}

.orb {
  position: fixed;
  border-radius: 50%;
  filter: blur(100px);
  pointer-events: none;
}

.orb-1 {
  width: 500px;
  height: 500px;
  background: rgba(249, 115, 22, 0.1);
  top: -150px;
  right: -100px;
}

.orb-2 {
  width: 400px;
  height: 400px;
  background: rgba(100, 200, 255, 0.05);
  bottom: -100px;
  left: -100px;
}

/* Card */
.quiz-card {
  background: rgba(15, 15, 25, 0.9);
  border: 1px solid rgba(255, 255, 255, 0.08);
  backdrop-filter: blur(20px);
  width: 100%;
  max-width: 720px;
  padding: 48px;
  border-radius: 28px;
  position: relative;
  z-index: 1;
  box-shadow:
    0 0 0 1px rgba(249, 115, 22, 0.05),
    0 40px 80px rgba(0, 0, 0, 0.6),
    inset 0 1px 0 rgba(255, 255, 255, 0.05);
}

/* Back button */
.back-button {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 28px;
  text-decoration: none;
  font-size: 13px;
  font-weight: 600;
  color: #475569;
  letter-spacing: 0.04em;
  transition: all 0.2s ease;
  padding: 8px 14px;
  border-radius: 10px;
  border: 1px solid transparent;
}

.back-button:hover {
  color: #f97316;
  background: rgba(249, 115, 22, 0.08);
  border-color: rgba(249, 115, 22, 0.2);
  transform: translateX(-3px);
}

/* Confirm Modal */
.confirm-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.8);
  backdrop-filter: blur(8px);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.confirm-box {
  background: #0f0f1a;
  border: 1px solid rgba(255, 255, 255, 0.1);
  padding: 40px 36px;
  border-radius: 24px;
  text-align: center;
  width: 360px;
  box-shadow:
    0 0 0 1px rgba(249, 115, 22, 0.15),
    0 40px 80px rgba(0, 0, 0, 0.8);
}

.confirm-icon {
  font-size: 40px;
  margin-bottom: 16px;
  display: block;
}

.confirm-title {
  font-family: "Bebas Neue", cursive;
  font-size: 32px;
  color: #ffffff;
  margin: 0 0 10px;
  letter-spacing: 0.05em;
}

.confirm-desc {
  font-size: 14px;
  color: #64748b;
  margin: 0 0 28px;
  line-height: 1.5;
}

.confirm-actions {
  display: flex;
  gap: 12px;
  justify-content: center;
}

.btn-confirm-yes {
  font-family: "Outfit", sans-serif;
  font-size: 14px;
  font-weight: 600;
  padding: 12px 24px;
  border-radius: 12px;
  border: none;
  cursor: pointer;
  background: linear-gradient(135deg, #f97316, #fb923c);
  color: #0a0a0f;
  box-shadow: 0 4px 16px rgba(249, 115, 22, 0.4);
  transition: all 0.2s ease;
}

.btn-confirm-yes:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 24px rgba(249, 115, 22, 0.5);
}

.btn-confirm-no {
  font-family: "Outfit", sans-serif;
  font-size: 14px;
  font-weight: 600;
  padding: 12px 24px;
  border-radius: 12px;
  border: 1px solid rgba(255, 255, 255, 0.1);
  cursor: pointer;
  background: rgba(255, 255, 255, 0.05);
  color: #94a3b8;
  transition: all 0.2s ease;
}

.btn-confirm-no:hover {
  background: rgba(255, 255, 255, 0.08);
  color: #e2e8f0;
}

/* Modal transition */
.modal-enter-active,
.modal-leave-active {
  transition: all 0.25s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-from .confirm-box,
.modal-leave-to .confirm-box {
  transform: scale(0.92) translateY(10px);
}
</style>
