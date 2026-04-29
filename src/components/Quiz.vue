<template>
  <div class="min-h-screen bg-white flex items-center justify-center p-4">
    <div class="w-full max-w-2xl">

      <!-- Loading State -->
      <div v-if="isLoading" class="flex flex-col items-center justify-center gap-4">
        <div class="w-12 h-12 border-4 border-purple-400 border-t-transparent rounded-full animate-spin"></div>
        <p class="text-gray-300 text-lg font-medium">Chargement du Tome {{ tomeNumber }}...</p>
      </div>

      <!-- Quiz Active -->
      <div v-else-if="!quizFinished" class="bg-white/10 backdrop-blur-md rounded-2xl shadow-2xl p-8 border border-white/10">

        <!-- Header -->
        <div class="mb-6">
          <div class="flex justify-between items-start mb-4">
            <div class="flex gap-4">
              <button
                @click="$emit('back')"
                class="p-2 bg-black/10 hover:bg-white/20 rounded-lg text-black transition-colors cursor-pointer"
                title="Retour à la liste"
              >
                <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                  <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18"></path>
                </svg>
              </button>
              <div>
                <span class="text-xs font-bold uppercase tracking-widest text-purple-500">Tome {{ tomeNumber }}</span>
                <h1 class="text-2xl font-bold text-black">Quiz Intelligent</h1>
              </div>
            </div>
            <span class="text-sm font-medium text-gray-900 bg-white/10 px-4 py-2 rounded-full">
              {{ currentIndex + 1 }} / {{ questions.length }}
            </span>
          </div>

          <!-- Progress bar -->
          <div class="w-full bg-white/10 rounded-full h-2.5 mb-2">
            <div
              class="bg-gradient-to-r from-purple-400 to-purple-500 h-2.5 rounded-full transition-all duration-500 ease-out"
              :style="{ width: progressPercent + '%' }"
            ></div>
          </div>
        </div>

        <!-- Question -->
        <div class="mb-8">
          <h2 class="text-xl font-semibold text-black mb-6 leading-relaxed">
            {{ currentQuestion.question }}
          </h2>

          <input
            v-model="userAnswer"
            @keyup.enter="validateAnswer"
            :disabled="isAnswerValidated"
            ref="answerInput"
            type="text"
            placeholder="Votre réponse..."
            class="w-full px-4 py-3 bg-white/10 border-2 border-white/20 rounded-xl focus:border-purple-400 focus:outline-none text-lg placeholder-gray-400 disabled:opacity-50 disabled:cursor-not-allowed transition-all duration-300"
          />
        </div>

        <!-- Feedback -->
        <div
          v-if="feedbackMessage"
          class="mb-6 p-4 rounded-xl transition-all duration-300 border-2"
          :class="isCorrect
            ? 'bg-emerald-500/20 border-emerald-400/50'
            : isSkipped
              ? 'bg-amber-500/20 border-amber-400/50'
              : 'bg-red-500/20 border-red-400/50'"
        >
          <p class="font-semibold text-lg mb-1" :class="isCorrect ? 'text-emerald-300' : isSkipped ? 'text-amber-300' : 'text-red-800'">
            {{ feedbackMessage }}
          </p>
          <p class="text-black">
            <span class="font-medium text-gray-800">Bonne réponse :</span> {{ currentQuestion.reponse }}
          </p>
        </div>

        <!-- Action Buttons -->
        <div class="flex flex-col sm:flex-row gap-3 w-full">
          <button
            @click="skipQuestion"
            :disabled="isAnswerValidated"
            class="flex-1 w-full bg-amber-500/20 text-amber-300 border border-amber-500/30 py-3 px-6 rounded-xl font-semibold hover:bg-amber-500/30 disabled:opacity-30 disabled:cursor-not-allowed transition-all duration-300 cursor-pointer"
          >
            Passer
          </button>

          <button
            @click="validateAnswer"
            :disabled="!userAnswer.trim() || isAnswerValidated"
            class="flex-1 w-full bg-purple-500 text-white py-3 px-6 rounded-xl font-semibold hover:from-purple-500 hover:to-pink-500 disabled:opacity-30 disabled:cursor-not-allowed transition-all duration-300 shadow-lg shadow-purple-500/20 cursor-pointer"
          >
            Valider
          </button>

          <button
            @click="nextQuestion"
            :disabled="!isAnswerValidated"
            class="flex-1 w-full bg-blue-500 text-white border border-white/20 py-3 px-6 rounded-xl font-semibold hover:bg-blue-300 disabled:opacity-30 disabled:cursor-not-allowed transition-all duration-300 cursor-pointer"
          >
            Suivant
          </button>
        </div>

      </div>

      <!-- Quiz Finished -->
      <div v-else class="bg-white/10 backdrop-blur-md rounded-2xl shadow-2xl p-8 text-center border border-white/10">
        <div class="mb-6">
          <div class="text-6xl mb-4">{{ getResultEmoji() }}</div>
          <h2 class="text-4xl font-bold text-white mb-2">Quiz Terminé !</h2>
          <span class="text-sm font-bold uppercase tracking-widest text-purple-400">Tome {{ tomeNumber }}</span>
        </div>

        <div class="bg-gradient-to-r from-purple-500/20 to-pink-500/20 rounded-xl p-6 mb-6 border border-purple-500/20">
          <p class="text-6xl font-black text-white mb-2">{{ score }} / {{ questions.length }}</p>
          <p class="text-2xl font-semibold text-purple-300">{{ successPercent }}% de réussite</p>
        </div>

        <div class="mb-4" v-if="isNewBestScore">
          <p class="text-lg text-yellow-400 font-bold">🏆 Nouveau meilleur score !</p>
        </div>

        <div class="mb-8">
          <p class="text-lg text-gray-300">{{ getEncouragementMessage() }}</p>
        </div>

        <div class="flex flex-col sm:flex-row gap-4 justify-center">
          <button
            @click="restartQuiz"
            class="bg-gradient-to-r from-purple-600 to-pink-600 text-white py-3 px-8 rounded-xl font-semibold hover:from-purple-500 hover:to-pink-500 transition-all duration-300 text-lg shadow-lg shadow-purple-500/20 cursor-pointer"
          >
            🔄 Rejouer ce tome
          </button>
          <button
            @click="$emit('back')"
            class="bg-white/10 text-white border border-white/20 py-3 px-8 rounded-xl font-semibold hover:bg-white/20 transition-all duration-300 text-lg cursor-pointer"
          >
            ← Retour aux tomes
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted, nextTick } from 'vue'
import { compareTwoStrings } from 'string-similarity'

const props = defineProps({
  tomeNumber: {
    type: Number,
    required: true
  },
  questionCount: {
    type: [Number, String],
    default: 20
  }
})

const emit = defineEmits(['back'])

// 🔀 Fonction pour mélanger les questions aléatoirement (algorithme de Fisher–Yates)
const shuffleArray = (array) => {
  const arr = [...array]
  for (let i = arr.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[arr[i], arr[j]] = [arr[j], arr[i]]
  }
  return arr
}

// Dynamic import mapping for all tomes
const tomeModules = import.meta.glob('../tome*.json')

const isLoading = ref(true)
const questions = ref([])
const currentIndex = ref(0)
const score = ref(0)
const userAnswer = ref('')
const isAnswerValidated = ref(false)
const feedbackMessage = ref('')
const isCorrect = ref(false)
const isSkipped = ref(false)
const quizFinished = ref(false)
const isNewBestScore = ref(false)
const answerInput = ref(null)

const currentQuestion = computed(() => questions.value[currentIndex.value])
const progressPercent = computed(() => ((currentIndex.value + 1) / questions.value.length) * 100)
const successPercent = computed(() => Math.round((score.value / questions.value.length) * 100))

// 📦 Chargement dynamique du tome sélectionné
onMounted(async () => {
  try {
    const modulePath = `../tome${props.tomeNumber}.json`
    const loader = tomeModules[modulePath]
    if (!loader) {
      console.error(`Tome ${props.tomeNumber} introuvable`)
      return
    }
    const module = await loader()
    let allQuestions = shuffleArray(module.default)

    // Limiter le nombre de questions si demandé
    if (props.questionCount !== 'all' && typeof props.questionCount === 'number') {
      allQuestions = allQuestions.slice(0, props.questionCount)
    }

    questions.value = allQuestions
  } catch (error) {
    console.error('Erreur lors du chargement du tome:', error)
  } finally {
    isLoading.value = false
    await nextTick()
    answerInput.value?.focus()
  }
})


// 🔤 Fonction pour normaliser et nettoyer la réponse
const normalizeString = (str) => {
  return str
    .toLowerCase()
    .normalize('NFD')
    .replace(/[\u0300-\u036f]/g, '') // Retire les accents
    .replace(/[^a-z0-9 ]/g, '') // Supprime la ponctuation
    .trim()
}

// 💡 Fonction pour comparer les réponses (tolérance aux fautes)
const validateAnswer = () => {
  if (!userAnswer.value.trim() || isAnswerValidated.value) return

  const userNormalized = normalizeString(userAnswer.value)
  const correctNormalized = normalizeString(currentQuestion.value.reponse)

  const similarity = compareTwoStrings(userNormalized, correctNormalized)

  // ✅ Conditions pour accepter la réponse :
  // - Similarité ≥ 0.5
  if (similarity >= 0.5) {
    isCorrect.value = true
    feedbackMessage.value = '✅ Bonne réponse !'
    score.value++
  } else {
    isCorrect.value = false
    feedbackMessage.value = '❌ Mauvaise réponse'
  }

  isSkipped.value = false
  isAnswerValidated.value = true
}

const nextQuestion = () => {
  if (currentIndex.value < questions.value.length - 1) {
    currentIndex.value++
    userAnswer.value = ''
    isAnswerValidated.value = false
    feedbackMessage.value = ''
    isCorrect.value = false
    isSkipped.value = false
    nextTick(() => answerInput.value?.focus())
  } else {
    quizFinished.value = true
    saveBestScore()
  }
}

const restartQuiz = () => {
  questions.value = shuffleArray(questions.value)
  currentIndex.value = 0
  score.value = 0
  userAnswer.value = ''
  isAnswerValidated.value = false
  feedbackMessage.value = ''
  isCorrect.value = false
  isSkipped.value = false
  quizFinished.value = false
  isNewBestScore.value = false
  nextTick(() => answerInput.value?.focus())
}

const skipQuestion = () => {
  // Ne valide pas la réponse, on saute simplement
  isAnswerValidated.value = true
  feedbackMessage.value = "⏭ Question passée"
  isCorrect.value = false
  isSkipped.value = true
}

// 💾 Sauvegarde du meilleur score dans localStorage
const saveBestScore = () => {
  const key = `quiz-best-score-tome${props.tomeNumber}`
  const previousBest = localStorage.getItem(key)
  const currentPercent = successPercent.value

  if (previousBest === null || currentPercent > parseInt(previousBest)) {
    localStorage.setItem(key, currentPercent.toString())
    isNewBestScore.value = true
  }
}

const getResultEmoji = () => {
  const percent = successPercent.value
  if (percent === 100) return '👑'
  if (percent >= 80) return '🎉'
  if (percent >= 60) return '💪'
  if (percent >= 40) return '📖'
  return '🔄'
}

const getEncouragementMessage = () => {
  const percent = successPercent.value
  if (percent === 100) return 'Parfait ! Vous êtes un expert !'
  if (percent >= 80) return 'Excellent travail ! Continue comme ça !'
  if (percent >= 60) return 'Bon travail ! Tu peux encore t\'améliorer.'
  if (percent >= 40) return 'Pas mal ! Encore quelques efforts.'
  return 'Ne te décourage pas ! Réessaie pour t\'améliorer.'
}
</script>