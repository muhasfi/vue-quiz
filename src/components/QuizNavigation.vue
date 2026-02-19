<script setup>
defineProps({
  currentQuestionIndex: Number,
  totalQuestions: Number,
  showResult: Boolean,
});

const emit = defineEmits(["next", "prev"]);
</script>

<template>
  <div v-if="!showResult">
    <button @click="emit('prev')" :disabled="currentQuestionIndex <= 0">
      Previous
    </button>

    <button
      @click="emit('next')"
      :disabled="currentQuestionIndex >= totalQuestions - 1"
    >
      Next
    </button>
  </div>
</template>
