<script setup>
import { ref, computed, onUnmounted } from 'vue'

// --- ESTADO (REATIVIDADE) ---
const timeInSeconds = ref(25 * 60) // 25 minutos
const isRunning = ref(false)
const mode = ref('Foco') // 'Foco' ou 'Pausa'
let intervalId = null // Variável normal, não precisa ser reativa pois não aparece na tela

// --- COMPUTED PROPERTIES ---
// Só recalculam se 'timeInSeconds' mudar.
const formattedTime = computed(() => {
  const minutes = Math.floor(timeInSeconds.value / 60)
  const seconds = timeInSeconds.value % 60
  // PadStart garante que 5 segundos vire "05"
  return `${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')}`
})

// --- FUNÇÕES DE CONTROLE ---
const startTimer = () => {
  if (isRunning.value) return
  
  isRunning.value = true
  intervalId = setInterval(() => {
    if (timeInSeconds.value > 0) {
      timeInSeconds.value--
    } else {
      // --- MUDANÇA AQUI ---
      pauseTimer()
      
      const audio = new Audio('/alarm.mp3')
      audio.play()
    
    }
  }, 1000)
}

const pauseTimer = () => {
  isRunning.value = false
  clearInterval(intervalId)
}

const resetTimer = () => {
  pauseTimer()
  // Reseta baseado no modo atual
  timeInSeconds.value = mode.value === 'Foco' ? 25 * 60 : 5 * 60
}

const switchMode = (newMode) => {
  mode.value = newMode
  resetTimer()
}

// --- CICLO DE VIDA ---
onUnmounted(() => {
  clearInterval(intervalId)
})
</script>

<template>
  <div class="app-container">
    <h1>FocusUp! 🍅</h1>
    
    <div class="timer-display">
      <h2 :class="mode === 'Foco' ? 'text-focus' : 'text-break'">
        {{ formattedTime }}
      </h2>
      <p>Modo: {{ mode }}</p>
    </div>

    <div class="controls">
      <button v-if="!isRunning" @click="startTimer">Iniciar</button>
      <button v-else @click="pauseTimer">Pausar</button>
      <button @click="resetTimer">Reiniciar</button>
    </div>

    <div class="modes">
      <button @click="switchMode('Foco')" :disabled="mode === 'Foco'">Modo Foco</button>
      <button @click="switchMode('Pausa')" :disabled="mode === 'Pausa'">Modo Pausa</button>
    </div>
  </div>
</template>

<style scoped>
.app-container {
  font-family: 'Segoe UI', sans-serif;
  text-align: center;
  max-width: 400px;
  margin: 60px auto;
  padding: 20px;
  background: #245f5c;
  color: #fff;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(35, 189, 122, 0.3);
}

.timer-display h2 {
  font-size: 4rem;
  margin: 20px 0;
}

.text-focus { color: #ff6b6b; } /* Vermelho suave */
.text-break { color: #4ecdc4; } /* Verde água */

button {
  margin: 5px;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  font-weight: bold;
  transition: transform 0.1s;
}

button:active { transform: scale(0.98); }
button:disabled { opacity: 0.5; cursor: not-allowed; }

.controls button { background: #fff; color: #005036; }
.modes button { background: #332004; color: #fff; font-size: 0.8rem; }
</style>