<template>
  <!-- Timer -->
  <div class="timer">
    Time Left: {{ timeLeft }}s
  </div>

  <!-- Landing Page -->
  <div
    v-if="showLanding"
    class="relative min-h-screen bg-cover bg-center text-center p-6"
  >
    <!-- Faded overlay -->
    <div class="absolute inset-0 bg-white opacity-85 pointer-events-none"></div>

    <!-- Landing content -->
    <div class="relative z-10 flex flex-col items-center justify-center min-h-screen text-center p-6">
      <img src="/ghana.svg" alt="Ghana Flag" class="w-32 h-auto mb-6" />
      <h1 class="text-4xl font-bold text-green-700 mb-4">🇬🇭 Test Your Knowledge of Ghana!</h1>
      <p class="text-lg text-gray-700 max-w-xl mb-6">
        Welcome to the <strong>Ultimate Ghana Quiz</strong>, a fun way to see how much you know about our beloved country!
      </p>
      <button
        @click="startAudio(); startQuiz()"
        class="px-6 py-3 rounded-lg bg-green-600 text-white font-semibold shadow-lg hover:bg-green-700 transition-all"
      >
        Start Quiz
      </button>
    </div>
  </div>

  <!-- Quiz Page -->
  <div
    v-else
    class="flex flex-col items-center justify-center min-h-screen bg-cover bg-center p-4"
  >
    <div class="w-full max-w-xl bg-white bg-opacity-90 rounded-2xl shadow-lg p-8">
      <!-- Progress Bar -->
      <div class="w-full h-3 bg-gray-200 rounded-full mb-6">
        <div
          class="h-3 bg-gradient-to-r from-red-500 via-yellow-400 to-green-600 rounded-full transition-all duration-300"
          :style="{ width: `${progress}%` }"
        ></div>
      </div>

      <!-- Quiz Content -->
      <div v-if="!showScore">
        <div class="mb-6 text-lg text-gray-800">
          Question {{ currentQuestionIndex + 1 }} of {{ questions.length }}
        </div>
        <h2 class="text-2xl font-bold mb-6 text-gray-900">{{ currentQuestion.question }}</h2>

        <div class="grid gap-4">
          <button
            v-for="(option, idx) in currentQuestion.options"
            :key="idx"
            @click="selectOption(idx)"
            :class="[
              'py-3 px-6 rounded-lg border text-lg font-medium transition-all duration-200',
              selectedOption === idx
                ? (idx === currentQuestion.answer
                    ? 'bg-green-500 text-white border-green-600 shadow-lg scale-105'
                    : 'bg-red-500 text-white border-red-600 shadow-lg scale-105')
                : (selectedOption !== null && idx === currentQuestion.answer
                    ? 'bg-green-500 text-white border-green-600 shadow-lg'
                    : 'bg-white text-gray-700 border-gray-300 hover:bg-yellow-100 hover:border-yellow-400')
            ]"
            :disabled="selectedOption !== null"
          >
            {{ option.text }}
          </button>
        </div>

        <div class="flex justify-between mt-8">
          <button
            class="px-4 py-2 rounded bg-gray-200 text-gray-600 font-semibold disabled:opacity-50"
            @click="prevQuestion"
            :disabled="currentQuestionIndex === 0"
          >
            Previous
          </button>
          <button
            class="px-4 py-2 rounded bg-blue-500 text-white font-semibold disabled:opacity-50"
            @click="nextQuestion"
            :disabled="selectedOption === null"
          >
            {{ isLastQuestion ? 'Finish' : 'Next' }}
          </button>
        </div>
      </div>

      <!-- Score Display -->
      <div v-else class="flex flex-col items-center justify-center min-h-[300px]">
        <h2 class="text-3xl font-bold mb-4 text-black">Quiz Complete!</h2>
        <p class="text-xl mb-6 text-gray-800">
          You scored <span class="font-bold text-blue-600">{{ score }}</span> out of
          <span class="font-bold text-purple-600">{{ questions.length }}</span>
        </p>
        <button
          class="px-6 py-3 rounded-lg bg-gradient-to-r from-green-600 to-yellow-400 text-white font-semibold shadow-lg hover:scale-105 transition-all"
          @click="restartQuiz"
        >
          Restart Quiz
        </button>
      </div>
    </div>
  </div>
</template>

<script>
import questions from '../assets/data/questions.json';
import correctSound from "../assets/correct.mp3";
import wrongSound from "../assets/wrong.mp3";
import ghanaAnthem from '../assets/gh.mp3';

const correctAudio = new Audio(correctSound);
const wrongAudio = new Audio(wrongSound);

export default {
  name: 'Quiz',
  data() {
    return {
      showLanding: true,
      questions,
      currentQuestionIndex: 0,
      selectedOption: null,
      userAnswers: [],
      showScore: false,
      score: 0,
      timeLeft: 30,
      timer: null,
      audio: null,
      audioPlaying: false
    }
  },
  computed: {
    currentQuestion() {
      return this.questions[this.currentQuestionIndex];
    },
    isLastQuestion() {
      return this.currentQuestionIndex === this.questions.length - 1;
    },
    progress() {
      return ((this.currentQuestionIndex + 1) / this.questions.length) * 100;
    }
  },
  methods: {
    // Start background audio
    startAudio() {
      if (!this.audio) {
        this.audio = new Audio(ghanaAnthem);
        this.audio.loop = true;
        this.audio.volume = 1.0;
      }
      this.audio.play().then(() => { this.audioPlaying = true; })
        .catch(err => console.error("Audio blocked:", err));
    },

    startQuiz() {
      this.showLanding = false;
      this.startTimer();
    },

    selectOption(idx) {
      this.selectedOption = idx;
      this.userAnswers[this.currentQuestionIndex] = idx;

      const option = this.currentQuestion.options[idx];
      if (option && option.isRight) {
        correctAudio.currentTime = 0;
        correctAudio.play();
      } else {
        wrongAudio.currentTime = 0;
        wrongAudio.play();
      }
    },

    nextQuestion() {
      if (this.isLastQuestion) {
        this.calculateScore();
        this.showScore = true;
        clearInterval(this.timer);
      } else {
        this.currentQuestionIndex++;
        this.selectedOption = this.userAnswers[this.currentQuestionIndex] ?? null;
        this.resetTimer();
      }
    },

    prevQuestion() {
      if (this.currentQuestionIndex > 0) {
        this.currentQuestionIndex--;
        this.selectedOption = this.userAnswers[this.currentQuestionIndex] ?? null;
        this.resetTimer();
      }
    },

    calculateScore() {
      this.score = this.questions.reduce((acc, question, index) => {
        const selectedIndex = this.userAnswers[index];
        const option = question.options[selectedIndex];
        return acc + (option && option.isRight ? 1 : 0);
      }, 0);
      console.log('Final Score:', this.score);
    },

    startTimer() {
      this.timeLeft = 30;
      this.timer = setInterval(() => {
        if (this.timeLeft > 0) {
          this.timeLeft--;
        } else {
          clearInterval(this.timer);
          this.nextQuestion();
        }
      }, 1000);
    },

    resetTimer() {
      clearInterval(this.timer);
      this.startTimer();
    },

    restartQuiz() {
      this.currentQuestionIndex = 0;
      this.selectedOption = null;
      this.userAnswers = [];
      this.showScore = false;
      this.score = 0;
    }
  },
  beforeUnmount() {
    if (this.audio) {
      this.audio.pause();
      this.audio = null;
    }
    clearInterval(this.timer);
  }
}
</script>

<style scoped>
</style>
