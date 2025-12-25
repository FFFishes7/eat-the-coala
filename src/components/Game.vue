<template>
  <div class="game">
    <!-- 经典模式：显示分数 -->
    <div v-if="mode === 'classic'" class="score-board">{{ score }}</div>
    
    <!-- 手速模式：显示倒计时 -->
    <div v-else class="timer-board">{{ countdown }}</div>

    <div class="render-layer">
      <div 
        v-for="(targetCol, rowIndex) in sequence" 
        :key="rowIndex"
        class="row"
        :class="{ 'bottom-row': rowIndex === 0 }"
      >
        <div 
          v-for="colIndex in 4" 
          :key="colIndex"
          class="cell"
          :class="{
            'coala': (colIndex - 1) === targetCol,
            'wrong': wrongColIndex !== -1 && rowIndex === 0 && (colIndex - 1) === wrongColIndex
          }"
        ></div>
      </div>
    </div>

    <div class="touch-layer">
      <div 
        v-for="(col, index) in 4" 
        :key="index"
        class="col-trigger"
        @touchstart.prevent="handleInput(index)"
        @mousedown.prevent="handleInput(index)"
      ></div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue';

const props = defineProps({
  mode: {
    type: String,
    default: 'classic'
  },
  timeLimit: {
    type: Number,
    default: 15
  }
});

const emit = defineEmits(['gameover']);

// --- 键盘映射 ---
const keyMap = {
  'g': 0,
  'h': 1,
  'j': 2,
  'k': 3
};

// --- 音效模块 ---
let audioCtx = null;

const initAudio = () => {
  if (!audioCtx) {
    audioCtx = new (window.AudioContext || window.webkitAudioContext)();
  }
};

// 点对音效：短促高音
const playTapSound = () => {
  if (!audioCtx) return;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  
  osc.frequency.value = 800;  // 频率
  osc.type = 'sine';
  gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.1);
  
  osc.start();
  osc.stop(audioCtx.currentTime + 0.1);
};

// 点错音效：低沉嗡声
const playErrorSound = () => {
  if (!audioCtx) return;
  const osc = audioCtx.createOscillator();
  const gain = audioCtx.createGain();
  osc.connect(gain);
  gain.connect(audioCtx.destination);
  
  osc.frequency.value = 150;  // 低频
  osc.type = 'square';
  gain.gain.setValueAtTime(0.3, audioCtx.currentTime);
  gain.gain.exponentialRampToValueAtTime(0.01, audioCtx.currentTime + 0.3);
  
  osc.start();
  osc.stop(audioCtx.currentTime + 0.3);
};

// --- 状态定义 ---
const sequence = ref([]);
const score = ref(0);
const wrongColIndex = ref(-1);
const countdown = ref(props.timeLimit);  // 倒计时
let inputLock = true;
let isOver = false;
let timerInterval = null;  // 计时器
let startTime = 0;         // 游戏开始时间戳
let hasStarted = false;    // 是否已开始（第一次点击）

// 计算已用时间（秒）
const getUsedTime = () => {
  return (Date.now() - startTime) / 1000;
};

// 启动计时
const startCountdown = () => {
  if (hasStarted) return;
  
  hasStarted = true;
  startTime = Date.now();
  
  // 仅手速模式启动倒计时
  if(props.mode === 'speed') {
    timerInterval = setInterval(() => {
    countdown.value--;
    if (countdown.value <= 0) {
      clearInterval(timerInterval);
      endGame();
    }
    }, 1000);
  }
};

// --- 游戏逻辑 ---

// 初始化
const initGame = () => {
  for (let i = 0; i < 8; i++) {
    addRandomRow();
  }

  // 300ms 防误触锁
  setTimeout(() => {
    inputLock = false;
  }, 300);
};

// 增加一行随机黑块
const addRandomRow = () => {
  sequence.value.push(Math.floor(Math.random() * 4));
};

// 核心输入处理
const handleInput = (colIndex) => {
  if (isOver || inputLock) return;

  const targetCol = sequence.value[0];

  // 第一次点击时启动计时和音频
  if (!hasStarted) {
    initAudio();
    startCountdown();
  }

  if (colIndex === targetCol) {
    playTapSound();
    sequence.value.shift();
    addRandomRow();
    score.value++;
  } else {
    playErrorSound();
    triggerGameOver(colIndex);
  }
};

// 游戏结束（点错）
const triggerGameOver = (index) => {
  isOver = true;
  wrongColIndex.value = index;
  if (timerInterval) clearInterval(timerInterval);  // 停止倒计时

  // 立即记录用时
  const finalTime = getUsedTime();

  if (navigator.vibrate) navigator.vibrate(200);

  setTimeout(() => {
    emit('gameover', { score: score.value, reason: 'wrong', time: finalTime });
  }, 300);
};

// 游戏结束（手速模式：时间到）
const endGame = () => {
  isOver = true;
  emit('gameover', { score: score.value, reason: 'timeout', time: props.timeLimit });
};

// 键盘事件处理
const handleKeydown = (e) => {
  if (e.repeat) return;
  
  const key = e.key.toLowerCase();
  if (key in keyMap) {
    handleInput(keyMap[key]);
  }
};

// 挂载时启动
onMounted(() => {
  initGame();
  window.addEventListener('keydown', handleKeydown);
});

// 卸载时清理
onUnmounted(() => {
  window.removeEventListener('keydown', handleKeydown);
  if (timerInterval) clearInterval(timerInterval);
});
</script>

<style scoped>
.game {
  position: absolute;
  inset: 0;
  background: #999;
}

/* 视觉层 */
.render-layer {
  position: absolute;
  inset: 0;
  display: flex;
  flex-direction: column-reverse;
  padding-bottom: env(safe-area-inset-bottom);
  z-index: 1;
}

.row {
  flex: 1;
  display: flex;
  background: #f7f7f7;
  border-top: 1px solid #ccc;
}

.row.bottom-row {
  border-bottom: 4px solid #4CAF50;
}

.cell {
  flex: 1;
  border-right: 1px solid #ccc;
  transition: background 0.05s;
}
.cell:last-child { border-right: none; }

.cell.coala { 
  background: url('/coala.png') center/contain no-repeat;
}
.cell.wrong { background: #ff4d4d !important; }

/* 触控层 */
.touch-layer {
  position: absolute;
  inset: 0;
  display: flex;
  z-index: 10;
  padding-bottom: env(safe-area-inset-bottom);
}

.col-trigger {
  flex: 1;
}

/* 分数 */
.score-board {
  position: absolute;
  top: 10%;
  width: 100%;
  text-align: center;
  font-size: 6rem;
  color: rgba(255, 60, 60, 0.3);
  font-weight: 900;
  pointer-events: none;
  z-index: 20;
}

/* 倒计时 */
.timer-board {
  position: absolute;
  top: 10%;
  width: 100%;
  text-align: center;
  font-size: 6rem;
  color: rgba(255, 152, 0, 0.5);
  font-weight: 900;
  pointer-events: none;
  z-index: 20;
}
</style>
