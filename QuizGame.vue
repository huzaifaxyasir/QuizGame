<template>
  <div class="quiz-container">
    <div class="background-pattern"></div>
    
    <div class="content-wrapper">
      <h1 class="quiz-title">🧠 Quiz Game</h1>

      <!-- Idle State -->
      <transition name="fade" mode="out-in">
        <div v-if="gameState === 'idle'" key="idle" class="state-container">
          <div class="welcome-card">
            <h2 class="welcome-text">Welcome to the Quiz Challenge!</h2>
            <p class="welcome-subtitle">Test your knowledge across various topics</p>
            <button @click="startQuiz" class="start-btn">
              <span class="btn-text">Start Quiz</span>
              <div class="btn-ripple"></div>
            </button>
          </div>
        </div>

        <!-- Question State -->
        <div v-else-if="gameState === 'question'" key="question" class="state-container">
          <div class="question-card">
            <div class="progress-bar">
              <div class="progress-fill" :style="{ width: progressPercentage + '%' }"></div>
            </div>
            
            <div class="question-header">
              <span class="question-number">Question {{ currentIndex + 1 }}/{{ questions.length }}</span>
              <div class="timer-container">
                <div class="timer-circle" :class="{ 'timer-warning': timer <= 3 }">
                  <span class="timer-text">{{ timer }}</span>
                </div>
              </div>
            </div>

            <transition name="slide-question" mode="out-in">
              <div :key="currentIndex" class="question-content">
                <p class="question-text">{{ currentQuestion.question }}</p>
                
                <ul class="options-list">
                  <li
                    v-for="(option, idx) in currentQuestion.options"
                    :key="idx"
                    @click="answerQuestion(option)"
                    class="option-item"
                    :style="{ '--delay': idx * 0.1 + 's' }"
                  >
                    <span class="option-letter">{{ String.fromCharCode(65 + idx) }}</span>
                    <span class="option-text">{{ option }}</span>
                    <div class="option-ripple"></div>
                  </li>
                </ul>
              </div>
            </transition>
          </div>
        </div>

        <!-- Result State -->
        <div v-else-if="gameState === 'result'" key="result" class="state-container">
          <div class="result-card">
            <div class="result-animation">
              <div class="confetti"></div>
              <div class="trophy">🏆</div>
            </div>
            
            <h2 class="result-title">Quiz Complete!</h2>
            <div class="score-display">
              <div class="score-circle">
                <span class="score-text">{{ score }}/{{ questions.length }}</span>
                <span class="score-percentage">{{ scorePercentage }}%</span>
              </div>
            </div>
            
            <div class="performance-message">
              <p class="performance-text">{{ performanceMessage }}</p>
            </div>
            
            <button @click="resetQuiz" class="restart-btn">
              <span class="btn-text">Try Again</span>
              <div class="btn-ripple"></div>
            </button>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onUnmounted } from 'vue'

const questions = [
  {
    question: 'What is the capital of France?',
    options: ['Paris', 'Berlin', 'Madrid', 'Rome'],
    answer: 'Paris',
  },
  {
    question: 'Which language runs in a web browser?',
    options: ['Java', 'C', 'Python', 'JavaScript'],
    answer: 'JavaScript',
  },
  {
    question: 'Who painted the Mona Lisa?',
    options: ['Van Gogh', 'Da Vinci', 'Picasso', 'Rembrandt'],
    answer: 'Da Vinci',
  },
  {
   question: 'What is the first capital of Pakistan?',
    options: ['Islamabad', 'Lahore', 'Quetta', 'Karachi'],
    answer: 'Karachi',
  },
  {
   question: 'What is the chemical symbol for gold?',
    options: ['Go', 'Gd', 'Au', 'Ag'],
    answer: 'Au',
  },
  {
   question: 'What is the capital of Australia?',
    options: ['Sydney', 'Melbourne', 'Canberra', 'Perth'],
    answer: 'Canberra',
  },
  {
    question: 'What is the hardest natural substance on Earth?',
    options: ['Gold', 'Diamond', 'Iron', 'Platinum'],
    answer: 'Diamond',
  },
  {
    question: 'Which gas makes up about 78% of Earth\'s atmosphere?',
    options: ['Oxygen', 'Carbon Dioxide', 'Nitrogen', 'Hydrogen'],
    answer: 'Nitrogen',
  },
   {
    question: 'What is the most spoken language in the world?',
    options: ['English', 'Spanish', 'Hindi', 'Mandarin Chinese'],
    answer: 'Mandarin Chinese',
  },
  {
    question: 'What is the currency of Japan?',
    options: ['Yen', 'Won', 'Yuan', 'Ringgit'],
    answer: 'Yen',
  },
]

const gameState = ref('idle')
const currentIndex = ref(0)
const score = ref(0)
const timer = ref(10)

let timerInterval = null

const currentQuestion = computed(() => questions[currentIndex.value])

const progressPercentage = computed(() => 
  ((currentIndex.value) / questions.length) * 100
)

const scorePercentage = computed(() => 
  Math.round((score.value / questions.length) * 100)
)

const performanceMessage = computed(() => {
  const percentage = scorePercentage.value
  if (percentage >= 90) return "Outstanding! You're a quiz master! 🎯"
  if (percentage >= 80) return "Excellent work! You know your stuff! 🌟"
  if (percentage >= 70) return "Great job! Well done! 👏"
  if (percentage >= 60) return "Good effort! Keep learning! 📚"
  if (percentage >= 50) return "Not bad! Room for improvement! 💪"
  return "Keep trying! Practice makes perfect! 🚀"
})

const startQuiz = () => {
  gameState.value = 'question'
  currentIndex.value = 0
  score.value = 0
  timer.value = 10
  startTimer()
}

const answerQuestion = (answer) => {
  if (answer === currentQuestion.value.answer) {
    score.value++
  }
  nextQuestion()
}

const nextQuestion = () => {
  stopTimer()
  currentIndex.value++
  
  if (currentIndex.value >= questions.length) {
    gameState.value = 'result'
  } else {
    timer.value = 10
    startTimer()
  }
}

const resetQuiz = () => {
  gameState.value = 'idle'
  stopTimer()
}

const startTimer = () => {
  stopTimer()
  timerInterval = setInterval(() => {
    timer.value--
    if (timer.value <= 0) {
      nextQuestion()
    }
  }, 1000)
}

const stopTimer = () => {
  if (timerInterval) {
    clearInterval(timerInterval)
    timerInterval = null
  }
}

onUnmounted(() => {
  stopTimer()
})
</script>

<style scoped>
* {
  box-sizing: border-box;
}

.quiz-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #FF6B6B 0%, #DC143C 50%, #8B0000 100%);
  position: relative;
  overflow: hidden;
  display: flex;
  align-items: center;
  justify-content: center;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
}

.background-pattern {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-image: 
    radial-gradient(circle at 25% 25%, rgba(255, 255, 255, 0.1) 2px, transparent 2px),
    radial-gradient(circle at 75% 75%, rgba(255, 255, 255, 0.05) 1px, transparent 1px);
  background-size: 50px 50px;
  animation: float 20s ease-in-out infinite;
}

@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

.content-wrapper {
  width: 100%;
  max-width: 600px;
  padding: 20px;
  z-index: 1;
}

.quiz-title {
  text-align: center;
  font-size: 3rem;
  font-weight: bold;
  color: white;
  margin-bottom: 2rem;
  text-shadow: 0 4px 8px rgba(0, 0, 0, 0.3);
  animation: pulse 2s ease-in-out infinite;
}

@keyframes pulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

.state-container {
  width: 100%;
}

/* Welcome Card */
.welcome-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 3rem 2rem;
  text-align: center;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.welcome-text {
  color: #DC143C;
  font-size: 2.2rem;
  margin-bottom: 1rem;
  font-weight: bold;
}

.welcome-subtitle {
  color: #666;
  font-size: 1.1rem;
  margin-bottom: 2rem;
}

/* Question Card */
.question-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 2rem;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.progress-bar {
  width: 100%;
  height: 6px;
  background: rgba(220, 20, 60, 0.2);
  border-radius: 3px;
  margin-bottom: 2rem;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #FF6B6B, #DC143C);
  border-radius: 3px;
  transition: width 0.5s ease;
}

.question-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.question-number {
  color: #DC143C;
  font-weight: bold;
  font-size: 1.1rem;
}

.timer-container {
  position: relative;
}

.timer-circle {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: linear-gradient(135deg, #FF6B6B, #DC143C);
  display: flex;
  align-items: center;
  justify-content: center;
  box-shadow: 0 4px 12px rgba(220, 20, 60, 0.3);
  transition: all 0.3s ease;
  animation: timerPulse 1s ease-in-out infinite;
}

.timer-warning {
  animation: timerWarning 0.5s ease-in-out infinite;
  background: linear-gradient(135deg, #FF4444, #CC0000);
}

@keyframes timerPulse {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.05); }
}

@keyframes timerWarning {
  0%, 100% { transform: scale(1); }
  50% { transform: scale(1.1); box-shadow: 0 0 20px rgba(255, 68, 68, 0.6); }
}

.timer-text {
  color: white;
  font-weight: bold;
  font-size: 1.5rem;
}

.question-content {
  text-align: center;
}

.question-text {
  font-size: 1.4rem;
  color: #333;
  margin-bottom: 2rem;
  font-weight: 600;
  line-height: 1.6;
}

.options-list {
  list-style: none;
  padding: 0;
  display: grid;
  gap: 1rem;
}

.option-item {
  background: linear-gradient(135deg, #FFF, #F8F8F8);
  border: 2px solid #E0E0E0;
  border-radius: 15px;
  padding: 1.2rem 1.5rem;
  cursor: pointer;
  display: flex;
  align-items: center;
  gap: 1rem;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  animation: slideInUp 0.6s ease forwards;
  animation-delay: var(--delay);
  opacity: 0;
  transform: translateY(30px);
}

@keyframes slideInUp {
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.option-item:hover {
  transform: translateY(-5px);
  border-color: #DC143C;
  box-shadow: 0 10px 25px rgba(220, 20, 60, 0.2);
  background: linear-gradient(135deg, #FFEBEE, #FFCDD2);
}

.option-letter {
  width: 40px;
  height: 40px;
  background: linear-gradient(135deg, #FF6B6B, #DC143C);
  color: white;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 1.1rem;
  flex-shrink: 0;
}

.option-text {
  font-size: 1.1rem;
  color: #333;
  font-weight: 500;
}

.option-ripple {
  position: absolute;
  border-radius: 50%;
  background: rgba(220, 20, 60, 0.3);
  transform: scale(0);
  animation: ripple 0.6s linear;
  pointer-events: none;
}

.option-item:active .option-ripple {
  animation: ripple 0.6s linear;
}

@keyframes ripple {
  to {
    transform: scale(4);
    opacity: 0;
  }
}

/* Result Card */
.result-card {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 20px;
  padding: 3rem 2rem;
  text-align: center;
  box-shadow: 0 20px 40px rgba(0, 0, 0, 0.2);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255, 255, 255, 0.3);
}

.result-animation {
  position: relative;
  margin-bottom: 2rem;
}

.trophy {
  font-size: 4rem;
  animation: bounce 2s ease-in-out infinite;
}

@keyframes bounce {
  0%, 20%, 50%, 80%, 100% { transform: translateY(0); }
  40% { transform: translateY(-10px); }
  60% { transform: translateY(-5px); }
}

.confetti {
  position: absolute;
  top: -20px;
  left: 50%;
  transform: translateX(-50%);
  width: 100px;
  height: 100px;
  background: linear-gradient(45deg, #FF6B6B, #FFD93D, #6BCF7F, #4D96FF);
  border-radius: 50%;
  animation: confetti 3s ease-in-out infinite;
  opacity: 0.8;
}

@keyframes confetti {
  0%, 100% { transform: translateX(-50%) rotate(0deg) scale(0); }
  50% { transform: translateX(-50%) rotate(180deg) scale(1); }
}

.result-title {
  color: #DC143C;
  font-size: 2.5rem;
  margin-bottom: 2rem;
  font-weight: bold;
}

.score-display {
  margin-bottom: 2rem;
}

.score-circle {
  width: 150px;
  height: 150px;
  border-radius: 50%;
  background: linear-gradient(135deg, #FF6B6B, #DC143C);
  color: white;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  margin: 0 auto;
  box-shadow: 0 10px 30px rgba(220, 20, 60, 0.3);
  animation: scoreReveal 1s ease-out;
}

@keyframes scoreReveal {
  0% { transform: scale(0) rotate(180deg); }
  100% { transform: scale(1) rotate(0deg); }
}

.score-text {
  font-size: 2rem;
  font-weight: bold;
}

.score-percentage {
  font-size: 1rem;
  opacity: 0.9;
}

.performance-message {
  margin-bottom: 2rem;
}

.performance-text {
  font-size: 1.2rem;
  color: #666;
  font-weight: 500;
}

/* Buttons */
.start-btn, .restart-btn {
  background: linear-gradient(135deg, #FF6B6B, #DC143C);
  color: white;
  border: none;
  padding: 1rem 3rem;
  border-radius: 50px;
  font-size: 1.2rem;
  font-weight: bold;
  cursor: pointer;
  position: relative;
  overflow: hidden;
  transition: all 0.3s ease;
  box-shadow: 0 8px 25px rgba(220, 20, 60, 0.3);
}

.start-btn:hover, .restart-btn:hover {
  transform: translateY(-3px);
  box-shadow: 0 12px 35px rgba(220, 20, 60, 0.4);
}

.start-btn:active, .restart-btn:active {
  transform: translateY(0);
}

.btn-text {
  position: relative;
  z-index: 1;
}

.btn-ripple {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 0;
  height: 0;
  border-radius: 50%;
  background: rgba(255, 255, 255, 0.3);
  transform: translate(-50%, -50%);
  transition: width 0.6s, height 0.6s;
}

.start-btn:active .btn-ripple,
.restart-btn:active .btn-ripple {
  width: 300px;
  height: 300px;
}

/* Transitions */
.fade-enter-active, .fade-leave-active {
  transition: all 0.5s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: scale(0.9);
}

.slide-question-enter-active, .slide-question-leave-active {
  transition: all 0.4s ease;
}

.slide-question-enter-from {
  opacity: 0;
  transform: translateX(50px);
}

.slide-question-leave-to {
  opacity: 0;
  transform: translateX(-50px);
}

/* Responsive Design */
@media (max-width: 768px) {
  .quiz-title {
    font-size: 2.5rem;
  }
  
  .content-wrapper {
    padding: 10px;
  }
  
  .welcome-card, .question-card, .result-card {
    padding: 2rem 1.5rem;
  }
  
  .question-text {
    font-size: 1.2rem;
  }
  
  .option-item {
    padding: 1rem;
  }
  
  .timer-circle {
    width: 50px;
    height: 50px;
  }
  
  .score-circle {
    width: 120px;
    height: 120px;
  }
}
</style>