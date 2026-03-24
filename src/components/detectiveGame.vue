<script setup>
import { ref, reactive, computed } from 'vue'

// 1. CONTEXTO DE LA HISTORIA
const intro = "During the annual autumn gala, the library remained open to the guests. While the music played in the ballroom, a sophisticated theft was taking place. It wasn't until the following morning, when the sunlight hit the display case, that the deception was noticed.";

// 2. ESTRUCTURA CON RESPUESTAS CORRECTAS
const storySegments = [
  { text: "The display case ", gapId: 1, correctAnswer: "was opened", after: " with a master key, leaving no marks on the glass. " },
  { text: "Inside, the original manuscript ", gapId: 2, correctAnswer: "was replaced", after: " by a clever forgery. " },
  { text: "The security logs ", gapId: 3, correctAnswer: "had been disabled", after: " manually, so no alarms were triggered during the gala. " },
  { text: "However, a small smudge of blue ink ", gapId: 4, correctAnswer: "was discovered", after: " on the corner of the pedestal. " },
  { text: "It is believed that the thief ", gapId: 5, correctAnswer: "was assisted", after: " by someone inside the house. " },
  { text: "Now, every guest ", gapId: 6, correctAnswer: "is being questioned", after: " by the police until the truth is found." }
];

// 3. OPCIONES DISPONIBLES
const options = ref([
  'was opened', 
  'was replaced', 
  'had been disabled', 
  'was discovered', 
  'was assisted', 
  'is being questioned'
]);

// 4. ESTADOS REACTIVOS
const userAnswers = reactive({});
const showVerdict = ref(false);
const isAllCorrect = ref(false);

// 5. LÓGICA DE JUEGO (FUNCIONES)
const handleDragStart = (word, event) => {
  event.dataTransfer.setData('word', word);
}

const handleDrop = (gapId, event) => {
  const word = event.dataTransfer.getData('word');
  userAnswers[gapId] = word;
  showVerdict.value = false; // Escondemos el veredicto si el usuario cambia algo
}

const isComplete = computed(() => {
  return Object.keys(userAnswers).length === storySegments.length;
});

const analyzeCase = () => {
  const finalCheck = storySegments.every(segment => 
    userAnswers[segment.gapId] === segment.correctAnswer
  );
  isAllCorrect.value = finalCheck;
  showVerdict.value = true;
};

const checkIndividualAnswer = (gapId) => {
  const segment = storySegments.find(s => s.gapId === gapId);
  return userAnswers[gapId] === segment.correctAnswer;
};

const resetGame = () => {
  showVerdict.value = false;
  isAllCorrect.value = false;
  // Limpiamos el objeto de respuestas
  Object.keys(userAnswers).forEach(key => delete userAnswers[key]);
};
</script>

<template>
  <div class="space-y-12 animate-fade-in max-w-4xl mx-auto">
    
    <section class="border-l-4 border-slate-300 pl-6 py-2 italic text-slate-600 leading-relaxed">
      <p>{{ intro }}</p>
    </section>

    <section>
      <h3 class="text-xs font-bold uppercase tracking-[0.3em] text-slate-400 mb-6">Collected Evidence:</h3>
      <div class="flex flex-wrap gap-3">
        <div 
          v-for="word in options" 
          :key="word" 
          draggable="true"
          @dragstart="handleDragStart(word, $event)"
          class="cursor-grab active:cursor-grabbing px-4 py-2 bg-white border border-slate-300 text-slate-700 font-mono text-sm hover:bg-slate-800 hover:text-white transition-all duration-300 shadow-sm"
        >
          {{ word }}
        </div>
      </div>
    </section>

    <section class="bg-stone-50 p-8 rounded-sm border border-stone-200 shadow-inner relative">
      <div class="absolute -top-4 right-10 w-8 h-12 border-2 border-slate-400 rounded-full opacity-30"></div>
      
      <div class="prose prose-slate max-w-none">
        <p class="text-xl leading-[2.8rem] text-slate-800 font-serif">
          <span v-for="segment in storySegments" :key="segment.gapId">
            {{ segment.text }}
            <span 
              @dragover.prevent 
              @drop="handleDrop(segment.gapId, $event)"
              class="inline-block min-w-[150px] border-b-2 border-dotted border-slate-400 mx-1 text-center transition-all px-2"
              :class="{
                'text-blue-700 font-bold italic': userAnswers[segment.gapId] && !showVerdict,
                'text-green-600 font-bold': showVerdict && checkIndividualAnswer(segment.gapId),
                'text-red-500 font-bold line-through': showVerdict && !checkIndividualAnswer(segment.gapId)
              }"
            >
              {{ userAnswers[segment.gapId] || '...............' }}
            </span>
            {{ segment.after }}
          </span>
        </p>
      </div>
    </section>

    <div class="flex flex-col items-center gap-8 pb-20">
      
      <button 
        @click="analyzeCase"
        :disabled="!isComplete || (showVerdict && isAllCorrect)"
        class="group relative px-12 py-4 bg-slate-900 text-white overflow-hidden transition-all disabled:opacity-30 disabled:cursor-not-allowed shadow-2xl"
      >
        <span class="relative z-10 font-bold tracking-[0.2em] uppercase text-xs">Analyze Case File</span>
        <div class="absolute inset-0 bg-blue-700 translate-y-full group-hover:translate-y-0 transition-transform duration-300"></div>
      </button>

      <div v-if="showVerdict" class="w-full max-w-2xl animate-fade-in">
        
        <div v-if="isAllCorrect" class="bg-green-50 border-2 border-green-600 p-8 text-center shadow-lg">
          <h2 class="text-2xl font-serif font-bold text-green-800 mb-2">CASE SOLVED! 🔍</h2>
          <p class="text-green-700">Excellent! The passive voice structures are correct. The mystery is over.</p>
        </div>

        <div v-else class="bg-red-50 border-2 border-red-600 p-8 text-center shadow-lg">
          <h2 class="text-2xl font-serif font-bold text-red-800 mb-2">INVESTIGATION FAILED</h2>
          <p class="text-red-700 mb-4">The clues don't match. Look at the red marks in the report and try again.</p>
          <button @click="resetGame" class="text-xs font-bold underline uppercase tracking-widest text-red-900 hover:text-black transition-colors">
            Reset Evidence
          </button>
        </div>

      </div>

      <p v-if="!isComplete" class="text-[10px] text-slate-400 uppercase tracking-[0.2em]">
        Fill all evidence gaps to proceed
      </p>
    </div>
  </div>
</template>

<style scoped>
.animate-fade-in {
  animation: fadeIn 0.8s ease-out forwards;
}

@keyframes fadeIn {
  from { opacity: 0; transform: translateY(10px); }
  to { opacity: 1; transform: translateY(0); }
}
</style>
