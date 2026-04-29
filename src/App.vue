<script setup>
import { ref } from 'vue'
import TomeSelector from './components/TomeSelector.vue'
import Quiz from './components/Quiz.vue'

const currentScreen = ref('selector') // 'selector' or 'quiz'
const selectedTome = ref(null)
const selectedCount = ref(20)

// Unique key to force Quiz component re-creation on tome change
const quizKey = ref(0)

const startQuiz = ({ tomeNumber, questionCount }) => {
  selectedTome.value = tomeNumber
  selectedCount.value = questionCount
  quizKey.value++
  currentScreen.value = 'quiz'
}

const backToSelector = () => {
  currentScreen.value = 'selector'
  selectedTome.value = null
}
</script>

<template>
  <Transition name="fade" mode="out-in">
    <TomeSelector
      v-if="currentScreen === 'selector'"
      @select="startQuiz"
      key="selector"
    />
    <Quiz
      v-else
      :key="quizKey"
      :tome-number="selectedTome"
      :question-count="selectedCount"
      @back="backToSelector"
    />
  </Transition>
</template>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.35s ease;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
