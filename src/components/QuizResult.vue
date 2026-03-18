<script setup>
import { computed, ref } from "vue";

const { quizQuestionsLength, numberOfCorrectAnswer, questions, userAnswers } =
  defineProps([
    "quizQuestionsLength",
    "numberOfCorrectAnswer",
    "questions",
    "userAnswers",
  ]);

const showDrawer = ref(false);
const filter = ref("all"); // "all" | "wrong" | "correct"

const percentage = computed(() =>
  Math.round((numberOfCorrectAnswer / quizQuestionsLength) * 100),
);

const grade = computed(() => {
  if (percentage.value >= 80)
    return {
      label: "Luar Biasa!",
      emoji: "🏆",
      color: "#f97316",
      glow: "rgba(249,115,22,0.3)",
    };
  if (percentage.value >= 60)
    return {
      label: "Bagus!",
      emoji: "👍",
      color: "#22d3ee",
      glow: "rgba(34,211,238,0.3)",
    };
  if (percentage.value >= 40)
    return {
      label: "Lumayan!",
      emoji: "💪",
      color: "#fbbf24",
      glow: "rgba(251,191,36,0.3)",
    };
  return {
    label: "Ayo Coba Lagi!",
    emoji: "🔄",
    color: "#ef4444",
    glow: "rgba(239,68,68,0.3)",
  };
});

const reviewList = computed(() => {
  return questions.map((question, index) => {
    const correctAnswer = question.answers.find((a) => a.correct);
    const userAnswerId = userAnswers[index];
    const userAnswer = question.answers.find((a) => a.id === userAnswerId);
    const isCorrect = userAnswerId === correctAnswer?.id;
    return {
      number: index + 1,
      text: question.text,
      isCorrect,
      userAnswer: userAnswer
        ? `${userAnswer.label}. ${userAnswer.text}`
        : "Tidak dijawab",
      correctAnswer: correctAnswer
        ? `${correctAnswer.label}. ${correctAnswer.text}`
        : "-",
    };
  });
});

const filteredReview = computed(() => {
  if (filter.value === "wrong")
    return reviewList.value.filter((r) => !r.isCorrect);
  if (filter.value === "correct")
    return reviewList.value.filter((r) => r.isCorrect);
  return reviewList.value;
});

const wrongCount = computed(
  () => reviewList.value.filter((r) => !r.isCorrect).length,
);
</script>

<template>
  <section class="results">
    <!-- Top badge -->
    <div
      class="grade-badge"
      :style="{
        color: grade.color,
        borderColor: grade.color + '33',
        background: grade.color + '11',
      }"
    >
      <span>{{ grade.emoji }}</span>
      <span class="grade-label">{{ grade.label }}</span>
    </div>

    <!-- Score ring -->
    <div class="score-ring-wrapper">
      <div class="ring-outer" :style="{ boxShadow: `0 0 60px ${grade.glow}` }">
        <svg viewBox="0 0 140 140" class="ring-svg">
          <circle cx="70" cy="70" r="58" class="ring-bg" />
          <circle
            cx="70"
            cy="70"
            r="58"
            class="ring-fill"
            :style="{
              strokeDasharray: `${(364 * percentage) / 100} 364`,
              stroke: grade.color,
            }"
          />
        </svg>
        <div class="score-center">
          <span class="score-num" :style="{ color: grade.color }">{{
            numberOfCorrectAnswer
          }}</span>
          <span class="score-sep">/{{ quizQuestionsLength }}</span>
        </div>
      </div>
    </div>

    <!-- Percentage -->
    <div class="pct-display">
      <span class="pct-num" :style="{ color: grade.color }">{{
        percentage
      }}</span>
      <span class="pct-symbol">%</span>
    </div>
    <p class="pct-label">jawaban benar</p>

    <!-- Stats row -->
    <div class="stats-row">
      <div class="stat-box correct">
        <span class="stat-val">{{ numberOfCorrectAnswer }}</span>
        <span class="stat-name">Benar</span>
      </div>
      <div class="stat-sep"></div>
      <div class="stat-box wrong">
        <span class="stat-val">{{
          quizQuestionsLength - numberOfCorrectAnswer
        }}</span>
        <span class="stat-name">Salah</span>
      </div>
      <div class="stat-sep"></div>
      <div class="stat-box total">
        <span class="stat-val">{{ quizQuestionsLength }}</span>
        <span class="stat-name">Total</span>
      </div>
    </div>
    <!-- Trigger drawer -->
    <button class="review-trigger-btn" @click="showDrawer = true">
      <svg
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2"
        width="15"
        height="15"
      >
        <path d="M9 11l3 3L22 4" />
        <path d="M21 12v7a2 2 0 01-2 2H5a2 2 0 01-2-2V5a2 2 0 012-2h11" />
      </svg>
      Lihat Review Jawaban
    </button>

    <!-- Overlay -->
    <Transition name="fade">
      <div
        v-if="showDrawer"
        class="drawer-overlay"
        @click="showDrawer = false"
      />
    </Transition>

    <!-- Drawer -->
    <Transition name="drawer">
      <div v-if="showDrawer" class="review-drawer">
        <!-- Handle -->
        <!-- <div class="drawer-handle-wrap">
          <div class="drawer-handle"></div>
        </div> -->

        <!-- Header -->
        <div class="drawer-header">
          <div>
            <div class="drawer-title">Review Jawaban</div>
            <div class="drawer-subtitle">
              {{ wrongCount }} soal perlu diperhatikan
            </div>
          </div>
          <div class="drawer-actions">
            <button
              class="filter-pill"
              :class="{ active: filter === 'all' }"
              @click="filter = 'all'"
            >
              Semua
            </button>
            <button
              class="filter-pill filter-correct"
              :class="{ active: filter === 'correct' }"
              @click="filter = 'correct'"
            >
              Benar
            </button>
            <button
              class="filter-pill filter-wrong"
              :class="{ active: filter === 'wrong' }"
              @click="filter = 'wrong'"
            >
              Salah
            </button>
            <button class="drawer-close" @click="showDrawer = false">✕</button>
          </div>
        </div>

        <!-- List -->
        <div class="drawer-list">
          <div
            v-for="item in filteredReview"
            :key="item.number"
            class="review-card"
            :class="item.isCorrect ? 'card-correct' : 'card-wrong'"
          >
            <div class="card-header">
              <span class="card-num">Soal {{ item.number }}</span>
              <span
                class="card-badge"
                :class="item.isCorrect ? 'badge-correct' : 'badge-wrong'"
              >
                {{ item.isCorrect ? "✓ Benar" : "✗ Salah" }}
              </span>
            </div>
            <p class="card-question">{{ item.text }}</p>
            <div v-if="!item.isCorrect" class="answer-row answer-user">
              <span class="answer-label">Jawaban kamu</span>
              <span class="answer-text wrong-text">{{ item.userAnswer }}</span>
            </div>
            <div class="answer-row answer-correct">
              <span class="answer-label">Jawaban benar</span>
              <span class="answer-text correct-text">{{
                item.correctAnswer
              }}</span>
            </div>
          </div>
        </div>
      </div>
    </Transition>

    <RouterLink to="/" class="back-btn">
      <svg
        viewBox="0 0 24 24"
        fill="none"
        stroke="currentColor"
        stroke-width="2.5"
        width="15"
        height="15"
      >
        <path d="M3 9l9-7 9 7v11a2 2 0 01-2 2H5a2 2 0 01-2-2z" />
        <polyline points="9 22 9 12 15 12 15 22" />
      </svg>
      Kembali ke Home
    </RouterLink>
  </section>
</template>

<style scoped>
@import url("https://fonts.googleapis.com/css2?family=Bebas+Neue&family=Outfit:wght@400;500;600;700&display=swap");

.review-trigger-btn {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 12px 24px;
  background: rgba(255, 255, 255, 0.07);
  border: 1px solid rgba(255, 255, 255, 0.1);
  border-radius: 12px;
  color: #e2e8f0;
  font-family: "Outfit", sans-serif;
  font-size: 13px;
  font-weight: 600;
  cursor: pointer;
  margin-bottom: 16px;
  transition: all 0.2s ease;
}
.review-trigger-btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

/* Overlay */
.drawer-overlay {
  position: fixed;
  inset: 0;
  background: rgba(0, 0, 0, 0.6);
  z-index: 40;
}

/* Drawer */
.review-drawer {
  position: fixed;
  bottom: 0;
  left: 0;
  right: 0;
  background: #0f0f1a;
  border-top: 1px solid rgba(255, 255, 255, 0.08);
  border-radius: 20px 20px 0 0;
  z-index: 50;
  max-height: 75vh;
  display: flex;
  flex-direction: column;
}

/* .drawer-handle-wrap {
  display: flex;
  justify-content: center;
  padding: 12px 0 0;
} */
.drawer-handle {
  width: 36px;
  height: 4px;
  background: rgba(255, 255, 255, 0.15);
  border-radius: 2px;
}

.drawer-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 16px 20px 12px;
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
}
.drawer-title {
  font-size: 15px;
  font-weight: 700;
  color: #e2e8f0;
}
.drawer-subtitle {
  font-size: 12px;
  color: #475569;
  margin-top: 2px;
}
.drawer-actions {
  display: flex;
  align-items: center;
  gap: 8px;
}
.drawer-close {
  width: 28px;
  height: 28px;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.07);
  border: 1px solid rgba(255, 255, 255, 0.08);
  color: #64748b;
  cursor: pointer;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 13px;
}

/* Filter pills */
.filter-pill {
  padding: 5px 12px;
  border-radius: 100px;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  font-family: "Outfit", sans-serif;
  background: rgba(255, 255, 255, 0.07);
  color: #64748b;
  border: 1px solid rgba(255, 255, 255, 0.08);
  transition: all 0.2s ease;
}
.filter-pill.active {
  background: #f97316;
  color: #0a0a0f;
  border-color: transparent;
}
.filter-wrong.active {
  background: #f87171;
  color: #0a0a0f;
}
.filter-correct.active {
  background: #4ade80;
  color: #0a0a0f;
}

/* Scrollable list */
.drawer-list {
  overflow-y: auto;
  padding: 16px;
  display: flex;
  flex-direction: column;
  gap: 10px;
  flex: 1;
}

/* Review cards */
.review-card {
  border-radius: 12px;
  padding: 14px;
}
.card-correct {
  border: 1px solid rgba(74, 222, 128, 0.2);
  background: rgba(74, 222, 128, 0.05);
}
.card-wrong {
  border: 1px solid rgba(248, 113, 113, 0.2);
  background: rgba(248, 113, 113, 0.05);
}
.card-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}
.card-num {
  font-size: 11px;
  font-weight: 600;
  color: #475569;
  letter-spacing: 0.08em;
  text-transform: uppercase;
}
.card-badge {
  font-size: 11px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 100px;
}
.badge-correct {
  color: #4ade80;
  background: rgba(74, 222, 128, 0.1);
}
.badge-wrong {
  color: #f87171;
  background: rgba(248, 113, 113, 0.1);
}

.card-question {
  font-size: 13px;
  color: #cbd5e1;
  margin: 0 0 10px;
  line-height: 1.5;
}
.answer-row {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
  padding: 8px 12px;
  border-radius: 8px;
  margin-top: 6px;
}
.answer-user {
  background: rgba(248, 113, 113, 0.08);
}
.answer-correct {
  background: rgba(74, 222, 128, 0.08);
}
.answer-label {
  font-size: 11px;
  font-weight: 600;
  color: #475569;
  text-transform: uppercase;
  letter-spacing: 0.06em;
  white-space: nowrap;
  flex-shrink: 0;
}
.answer-text {
  font-size: 13px;
  font-weight: 500;
  text-align: right;
}
.wrong-text {
  color: #f87171;
}
.correct-text {
  color: #4ade80;
}

/* Transitions */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.drawer-enter-active,
.drawer-leave-active {
  transition: transform 0.35s cubic-bezier(0.32, 0.72, 0, 1);
}
.drawer-enter-from,
.drawer-leave-to {
  transform: translateY(100%);
}
/* Review section */
.review-section {
  width: 100%;
  max-width: 480px;
  margin-bottom: 32px;
}

.review-title {
  font-family: "Outfit", sans-serif;
  font-size: 12px;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #334155;
  margin: 0 0 16px;
  text-align: center;
}

.review-item {
  border-radius: 14px;
  border: 1px solid;
  padding: 16px;
  margin-bottom: 10px;
}

.review-correct {
  background: rgba(74, 222, 128, 0.05);
  border-color: rgba(74, 222, 128, 0.2);
}

.review-wrong {
  background: rgba(248, 113, 113, 0.05);
  border-color: rgba(248, 113, 113, 0.2);
}

.review-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.review-num {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.08em;
  text-transform: uppercase;
  color: #475569;
}

.review-badge {
  font-size: 11px;
  font-weight: 700;
  padding: 3px 10px;
  border-radius: 100px;
}

.review-correct .review-badge {
  color: #4ade80;
  background: rgba(74, 222, 128, 0.1);
}

.review-wrong .review-badge {
  color: #f87171;
  background: rgba(248, 113, 113, 0.1);
}

.review-question {
  font-size: 13px;
  font-weight: 500;
  color: #cbd5e1;
  margin: 0 0 12px;
  line-height: 1.5;
}

.review-answer {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 12px;
  padding: 8px 12px;
  border-radius: 8px;
  margin-top: 6px;
}

.review-answer-wrong {
  background: rgba(248, 113, 113, 0.08);
}

.review-answer-correct {
  background: rgba(74, 222, 128, 0.08);
}

.answer-label {
  font-size: 11px;
  font-weight: 600;
  letter-spacing: 0.06em;
  text-transform: uppercase;
  color: #475569;
  white-space: nowrap;
  flex-shrink: 0;
}

.review-answer-wrong .answer-text {
  color: #f87171;
}

.review-answer-correct .answer-text {
  color: #4ade80;
}

.answer-text {
  font-size: 13px;
  font-weight: 500;
  text-align: right;
}

.results {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 16px 0 8px;
  font-family: "Outfit", sans-serif;
}

.grade-badge {
  display: inline-flex;
  align-items: center;
  gap: 8px;
  font-size: 13px;
  font-weight: 600;
  letter-spacing: 0.06em;
  border: 1px solid;
  border-radius: 100px;
  padding: 8px 18px;
  margin-bottom: 32px;
}

.grade-label {
  text-transform: uppercase;
}

/* Ring */
.score-ring-wrapper {
  margin-bottom: 24px;
}

.ring-outer {
  border-radius: 50%;
  padding: 4px;
  position: relative;
}

.ring-svg {
  width: 160px;
  height: 160px;
  transform: rotate(-90deg);
  display: block;
}

.ring-bg {
  fill: none;
  stroke: rgba(255, 255, 255, 0.06);
  stroke-width: 10;
}

.ring-fill {
  fill: none;
  stroke-width: 10;
  stroke-linecap: round;
  transition: stroke-dasharray 1.2s cubic-bezier(0.4, 0, 0.2, 1);
}

.score-center {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}

.score-num {
  font-family: "Bebas Neue", cursive;
  font-size: 52px;
  line-height: 1;
  letter-spacing: 0.02em;
}

.score-sep {
  font-family: "Outfit", sans-serif;
  font-size: 14px;
  font-weight: 500;
  color: #334155;
  margin-top: 2px;
}

/* Percentage display */
.pct-display {
  display: flex;
  align-items: baseline;
  gap: 4px;
  margin-bottom: 6px;
}

.pct-num {
  font-family: "Bebas Neue", cursive;
  font-size: 64px;
  line-height: 1;
  letter-spacing: 0.02em;
}

.pct-symbol {
  font-family: "Bebas Neue", cursive;
  font-size: 32px;
  color: #475569;
}

.pct-label {
  font-size: 12px;
  font-weight: 500;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #334155;
  margin: 0 0 32px;
}

/* Stats row */
.stats-row {
  display: flex;
  align-items: center;
  gap: 24px;
  background: rgba(255, 255, 255, 0.03);
  border: 1px solid rgba(255, 255, 255, 0.07);
  border-radius: 16px;
  padding: 16px 32px;
  margin-bottom: 36px;
  width: 100%;
  max-width: 320px;
  justify-content: center;
}

.stat-box {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 4px;
}

.stat-val {
  font-family: "Bebas Neue", cursive;
  font-size: 28px;
  line-height: 1;
  letter-spacing: 0.05em;
}

.stat-name {
  font-size: 10px;
  font-weight: 600;
  letter-spacing: 0.12em;
  text-transform: uppercase;
  color: #334155;
}

.correct .stat-val {
  color: #4ade80;
}
.wrong .stat-val {
  color: #f87171;
}
.total .stat-val {
  color: #94a3b8;
}

.stat-sep {
  width: 1px;
  height: 40px;
  background: rgba(255, 255, 255, 0.06);
}

/* Back button */
.back-btn {
  display: inline-flex;
  align-items: center;
  gap: 10px;
  padding: 14px 28px;
  background: linear-gradient(135deg, #f97316, #fbbf24);
  color: #0a0a0f;
  font-family: "Outfit", sans-serif;
  font-size: 14px;
  font-weight: 700;
  border-radius: 14px;
  text-decoration: none;
  letter-spacing: 0.04em;
  box-shadow: 0 4px 20px rgba(249, 115, 22, 0.4);
  transition: all 0.2s ease;
}

.back-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 10px 32px rgba(249, 115, 22, 0.55);
}
.drawer-list::-webkit-scrollbar {
  width: 4px;
}

.drawer-list::-webkit-scrollbar-track {
  background: transparent;
}

.drawer-list::-webkit-scrollbar-thumb {
  background: #f97316;
  border-radius: 100px;
}

.drawer-list::-webkit-scrollbar-thumb:hover {
  background: #fbbf24;
}
</style>
