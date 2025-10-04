<template>
  <div class="min-h-screen bg-gradient-to-br from-indigo-100 via-purple-50 to-pink-100 py-6 flex flex-col justify-center font-inter">
    <div class="relative py-3 sm:max-w-xl sm:mx-auto w-full px-4 sm:px-0">
      <!-- Floating card with glass effect -->
      <div class="relative bg-white/80 backdrop-blur-lg shadow-2xl sm:rounded-3xl px-4 py-8 sm:p-12 border border-white/20">
        <!-- Decorative elements -->
        <div class="absolute -top-10 -left-10 w-32 h-32 bg-purple-300 rounded-full mix-blend-multiply filter blur-xl opacity-70 animate-blob"></div>
        <div class="absolute -bottom-8 -right-8 w-32 h-32 bg-yellow-300 rounded-full mix-blend-multiply filter blur-xl opacity-70 animate-blob animation-delay-2000"></div>
        <div class="absolute -bottom-8 -left-8 w-32 h-32 bg-pink-300 rounded-full mix-blend-multiply filter blur-xl opacity-70 animate-blob animation-delay-4000"></div>
        
        <div class="relative max-w-md mx-auto">
          <div class="flex items-center justify-between mb-8">
            <h1 class="text-4xl font-extrabold bg-clip-text text-transparent bg-gradient-to-r from-purple-600 to-pink-600">Pomodoro Timer</h1>
            <button 
              @click="toggleMute" 
              class="p-2 rounded-full hover:bg-black/5 transition-colors duration-200"
              :title="isMuted ? 'Unmute' : 'Mute'"
            >
              <SpeakerWaveIcon v-if="!isMuted" class="h-6 w-6 text-gray-600" />
              <SpeakerXMarkIcon v-else class="h-6 w-6 text-gray-600" />
            </button>
          </div>

          <!-- Timer Mode Selection -->
          <div class="flex justify-center space-x-2 mb-12">
            <button 
              @click="setMode('focus')" 
              :class="[
                'px-6 py-3 rounded-xl transition-all duration-300 shadow-sm',
                mode === 'focus' 
                  ? 'bg-gradient-to-r from-purple-500 to-purple-600 text-white shadow-purple-200' 
                  : 'bg-white/50 text-gray-600 hover:bg-white/80'
              ]"
            >
              <ClockIcon class="h-5 w-5 inline-block mr-2" />
              Focus
            </button>
            <button 
              @click="setMode('shortBreak')" 
              :class="[
                'px-6 py-3 rounded-xl transition-all duration-300 shadow-sm',
                mode === 'shortBreak' 
                  ? 'bg-gradient-to-r from-green-500 to-emerald-600 text-white shadow-green-200' 
                  : 'bg-white/50 text-gray-600 hover:bg-white/80'
              ]"
            >
              <BeakerIcon class="h-5 w-5 inline-block mr-2" />
              Short Break
            </button>
            <button 
              @click="setMode('longBreak')" 
              :class="[
                'px-6 py-3 rounded-xl transition-all duration-300 shadow-sm',
                mode === 'longBreak' 
                  ? 'bg-gradient-to-r from-blue-500 to-indigo-600 text-white shadow-blue-200' 
                  : 'bg-white/50 text-gray-600 hover:bg-white/80'
              ]"
            >
              <SunIcon class="h-5 w-5 inline-block mr-2" />
              Long Break
            </button>
          </div>

          <!-- Timer Display -->
          <div class="relative">
            <div class="absolute inset-0 bg-gradient-to-r from-violet-200 to-pink-200 rounded-full filter blur-3xl opacity-30"></div>
            <div class="relative text-8xl font-bold text-center mb-12 font-mono tracking-tight bg-clip-text text-transparent bg-gradient-to-r from-gray-900 to-gray-700">
              {{ formatTime }}
            </div>
          </div>

          <!-- Controls -->
          <div class="flex justify-center space-x-4 mb-12">
            <button 
              @click="startTimer" 
              :class="[
                'flex items-center px-8 py-4 rounded-xl transition-all duration-300 shadow-lg',
                isRunning
                  ? 'opacity-50 cursor-not-allowed'
                  : 'bg-gradient-to-r from-indigo-500 to-purple-600 text-white hover:translate-y-[-1px] hover:shadow-xl active:translate-y-[1px]'
              ]"
              :disabled="isRunning"
            >
              <PlayIcon class="h-5 w-5 mr-2" />
              Start
            </button>
            <button 
              @click="pauseTimer" 
              :class="[
                'flex items-center px-8 py-4 rounded-xl transition-all duration-300 shadow-lg',
                !isRunning
                  ? 'opacity-50 cursor-not-allowed'
                  : 'bg-gradient-to-r from-amber-500 to-yellow-600 text-white hover:translate-y-[-1px] hover:shadow-xl active:translate-y-[1px]'
              ]"
              :disabled="!isRunning"
            >
              <PauseIcon class="h-5 w-5 mr-2" />
              Pause
            </button>
            <button 
              @click="resetTimer" 
              class="flex items-center px-8 py-4 rounded-xl bg-gradient-to-r from-red-500 to-rose-600 text-white transition-all duration-300 shadow-lg hover:translate-y-[-1px] hover:shadow-xl active:translate-y-[1px]"
            >
              <ArrowPathIcon class="h-5 w-5 mr-2" />
              Reset
            </button>
          </div>

          <!-- Settings -->
          <div class="settings mt-8 bg-white/50 backdrop-blur-sm rounded-2xl p-6 shadow-lg border border-white/20">
            <div class="flex items-center justify-between mb-6">
              <h2 class="text-xl font-bold bg-clip-text text-transparent bg-gradient-to-r from-gray-900 to-gray-600">Settings</h2>
              <button 
                @click="isSettingsVisible = !isSettingsVisible"
                class="text-gray-500 hover:text-gray-700 transition-colors duration-200"
              >
                <ChevronUpIcon v-if="isSettingsVisible" class="h-6 w-6" />
                <ChevronDownIcon v-else class="h-6 w-6" />
              </button>
            </div>
            
            <div v-if="isSettingsVisible" 
                 class="space-y-6 transition-all duration-300"
                 :class="isSettingsVisible ? 'opacity-100 transform translate-y-0' : 'opacity-0 transform -translate-y-4'"
            >
              <div class="flex items-center justify-between gap-4">
                <label class="text-gray-700 font-medium">Focus Time (minutes)</label>
                <input 
                  type="number" 
                  v-model="settings.focusTime" 
                  :class="[
                    'w-24 px-4 py-3 border rounded-xl bg-white/70 backdrop-blur-sm transition-all duration-200',
                    isRunning ? 'opacity-50 cursor-not-allowed' : 'focus:ring-2 focus:ring-purple-500 focus:border-purple-500 hover:bg-white'
                  ]"
                  :disabled="isRunning"
                  min="1"
                >
              </div>
              <div class="flex items-center justify-between gap-4">
                <label class="text-gray-700 font-medium">Short Break (minutes)</label>
                <input 
                  type="number" 
                  v-model="settings.shortBreak" 
                  :class="[
                    'w-24 px-4 py-3 border rounded-xl bg-white/70 backdrop-blur-sm transition-all duration-200',
                    isRunning ? 'opacity-50 cursor-not-allowed' : 'focus:ring-2 focus:ring-green-500 focus:border-green-500 hover:bg-white'
                  ]"
                  :disabled="isRunning"
                  min="1"
                >
              </div>
              <div class="flex items-center justify-between gap-4">
                <label class="text-gray-700 font-medium">Long Break (minutes)</label>
                <input 
                  type="number" 
                  v-model="settings.longBreak" 
                  :class="[
                    'w-24 px-4 py-3 border rounded-xl bg-white/70 backdrop-blur-sm transition-all duration-200',
                    isRunning ? 'opacity-50 cursor-not-allowed' : 'focus:ring-2 focus:ring-blue-500 focus:border-blue-500 hover:bg-white'
                  ]"
                  :disabled="isRunning"
                  min="1"
                >
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue'
import { 
  PlayIcon, 
  PauseIcon, 
  ArrowPathIcon,
  SpeakerWaveIcon,
  SpeakerXMarkIcon,
  ClockIcon,
  BeakerIcon,
  SunIcon,
  ChevronUpIcon,
  ChevronDownIcon
} from '@heroicons/vue/24/solid'

const settings = ref({
  focusTime: 25,
  shortBreak: 5,
  longBreak: 15
})

const timeLeft = ref(settings.value.focusTime * 60)
const isRunning = ref(false)
const timer = ref(null)
const mode = ref('focus')
const isMuted = ref(false)
const isSettingsVisible = ref(true)
const chime = ref(null)

onMounted(() => {
  // Create audio element for the chime
  chime.value = new Audio('https://assets.mixkit.co/active_storage/sfx/2869/2869-preview.mp3')
})

const formatTime = computed(() => {
  const minutes = Math.floor(timeLeft.value / 60)
  const seconds = timeLeft.value % 60
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})

const startTimer = () => {
  if (!isRunning.value) {
    isRunning.value = true
    timer.value = setInterval(() => {
      if (timeLeft.value > 0) {
        timeLeft.value--
      } else {
        // Time's up!
        clearInterval(timer.value)
        isRunning.value = false
        handleTimerComplete()
      }
    }, 1000)
  }
}

const handleTimerComplete = () => {
  if (!isMuted.value && chime.value) {
    chime.value.play()
  }
  
  // Auto switch modes
  if (mode.value === 'focus') {
    mode.value = 'shortBreak'
    timeLeft.value = settings.value.shortBreak * 60
  } else {
    mode.value = 'focus'
    timeLeft.value = settings.value.focusTime * 60
  }
}

const pauseTimer = () => {
  clearInterval(timer.value)
  isRunning.value = false
}

const resetTimer = () => {
  clearInterval(timer.value)
  isRunning.value = false
  setMode(mode.value)
}

const setMode = (newMode) => {
  mode.value = newMode
  switch (newMode) {
    case 'focus':
      timeLeft.value = settings.value.focusTime * 60
      break
    case 'shortBreak':
      timeLeft.value = settings.value.shortBreak * 60
      break
    case 'longBreak':
      timeLeft.value = settings.value.longBreak * 60
      break
  }
}

const toggleMute = () => {
  isMuted.value = !isMuted.value
}

// Watch for settings changes
watch(settings, (newSettings) => {
  if (!isRunning.value) {
    setMode(mode.value)
  }
}, { deep: true })
</script>

<style>
.font-inter {
  font-family: 'Inter', sans-serif;
}

@keyframes blob {
  0% {
    transform: translate(0px, 0px) scale(1);
  }
  33% {
    transform: translate(30px, -50px) scale(1.1);
  }
  66% {
    transform: translate(-20px, 20px) scale(0.9);
  }
  100% {
    transform: translate(0px, 0px) scale(1);
  }
}

.animate-blob {
  animation: blob 7s infinite;
}

.animation-delay-2000 {
  animation-delay: 2s;
}

.animation-delay-4000 {
  animation-delay: 4s;
}

button:disabled {
  cursor: not-allowed;
  opacity: 0.5;
}

input[type="number"] {
  appearance: textfield;
  -moz-appearance: textfield;
}

input[type="number"]::-webkit-outer-spin-button,
input[type="number"]::-webkit-inner-spin-button {
  -webkit-appearance: none;
  appearance: none;
  margin: 0;
}
</style>