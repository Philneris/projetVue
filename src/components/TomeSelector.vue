<template>
  <div class="min-h-screen bg-white flex items-center justify-center p-4">
    <div class="w-full max-w-5xl">

      <!-- Header -->
      <div class="text-center mb-10">
        <h1 class="text-5xl font-extrabold text-transparent bg-clip-text bg-purple-400 mb-3 tracking-tight">
          Quiz Intelligent
        </h1>
        <p class="text-lg text-gray-900">Choisissez un tome pour commencer</p>
      </div>

      <!-- Question Count Selector -->
      <div class="flex justify-center mb-8">
        <div class="bg-white/10 backdrop-blur-md rounded-2xl p-4 border border-white/10 inline-flex items-center gap-4">
          <span class="text-gray-900 font-medium text-sm">Nombre de questions :</span>
          <div class="flex gap-2">
            <button
              v-for="count in questionOptions"
              :key="count"
              @click="selectedCount = count"
              class="px-4 py-2 rounded-xl text-sm font-bold transition-all duration-300 cursor-pointer"
              :class="selectedCount === count
                ? 'bg-purple-500 text-white shadow-lg shadow-purple-500/30 scale-105'
                : 'bg-white/10 text-gray-900 hover:bg-purple-300 hover:text-gray-200'"
            >
              {{ count === 'all' ? 'Toutes' : count }}
            </button>
          </div>
        </div>
      </div>

      <!-- Tome Grid -->
      <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-3 sm:gap-5">
        <button
          v-for="tome in tomes"
          :key="tome.number"
          @click="selectTome(tome.number)"
          class="group relative overflow-hidden rounded-xl sm:rounded-2xl p-4 sm:p-6 text-left transition-all duration-500 hover:scale-[1.02] hover:shadow-xl cursor-pointer border border-white/10"
          :class="tome.bgClass"
        >
          <!-- Glow effect on hover -->
          <div
            class="absolute inset-0 opacity-0 group-hover:opacity-100 transition-opacity duration-500 rounded-xl sm:rounded-2xl"
            :class="tome.glowClass"
          ></div>

          <!-- Content -->
          <div class="relative z-10 flex sm:flex-col justify-between sm:justify-start items-center sm:items-start gap-4">
            <div class="flex items-center sm:items-start gap-3 sm:gap-0 sm:flex-col sm:mb-4">
              <span class="text-[10px] sm:text-xs font-bold uppercase tracking-widest text-white/60">Tome</span>
              <h2 class="text-2xl sm:text-4xl font-black text-white">{{ tome.number }}</h2>
            </div>

            <div class="flex flex-col sm:flex-row items-end sm:items-center justify-between sm:w-full">
              <span class="text-xs sm:text-sm text-white/70 font-medium whitespace-nowrap">
                {{ tome.count }} questions
              </span>
              <div v-if="tome.bestScore !== null" class="flex items-center gap-1.5 bg-black/20 rounded-full px-2 py-0.5 sm:px-3 sm:py-1 mt-1 sm:mt-0">
                <span class="text-yellow-400 text-[10px] sm:text-sm">⭐</span>
                <span class="text-white text-[10px] sm:text-xs font-bold">{{ tome.bestScore }}%</span>
              </div>
            </div>
          </div>

          <!-- Arrow indicator (hidden on mobile to save space) -->
          <div class="hidden sm:block absolute bottom-4 right-4 opacity-0 group-hover:opacity-100 transform translate-x-2 group-hover:translate-x-0 transition-all duration-300">
            <svg class="w-6 h-6 text-white/80" fill="none" stroke="currentColor" viewBox="0 0 24 24">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M13 7l5 5m0 0l-5 5m5-5H6"></path>
            </svg>
          </div>
        </button>
      </div>

    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const emit = defineEmits(['select'])

const selectedCount = ref(20)
const questionOptions = [10, 20, 50, 'all']

// Approximate question counts per tome (from file analysis)
const tomeQuestionCounts = {
  1: 1020, 2: 1000, 3: 996, 4: 760,
  5: 985, 6: 888, 7: 990, 8: 619, 9: 727
}

// Unique visual identity per tome
const tomeStyles = [
  { bgClass: 'bg-gradient-to-br from-violet-600/90 to-indigo-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(139,92,246,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-rose-600/90 to-pink-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(244,63,94,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-emerald-600/90 to-teal-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(16,185,129,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-amber-600/90 to-orange-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(245,158,11,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-cyan-600/90 to-blue-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(6,182,212,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-fuchsia-600/90 to-purple-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(192,38,211,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-lime-600/90 to-green-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(132,204,22,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-sky-600/90 to-indigo-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(14,165,233,0.3)]'},
  { bgClass: 'bg-gradient-to-br from-red-600/90 to-rose-800/90', glowClass: 'shadow-[inset_0_0_60px_rgba(239,68,68,0.3)]'},
]

const tomes = computed(() => {
  return Array.from({ length: 9 }, (_, i) => {
    const num = i + 1
    const bestScoreRaw = localStorage.getItem(`quiz-best-score-tome${num}`)
    return {
      number: num,
      count: tomeQuestionCounts[num],
      bestScore: bestScoreRaw !== null ? parseInt(bestScoreRaw) : null,
      ...tomeStyles[i]
    }
  })
})

const selectTome = (tomeNumber) => {
  emit('select', {
    tomeNumber,
    questionCount: selectedCount.value
  })
}
</script>
