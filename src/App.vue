<template>
  <div class="app-container">
    <!-- 状态机：根据 gameState 显示不同组件 -->
    <Home v-if="gameState === 'home'" @start="startGame" />
    <Game 
      v-else-if="gameState === 'playing'" 
      :mode="gameMode" 
      :timeLimit="timeLimit" 
      @gameover="handleGameOver" 
    />
    <Over 
      v-else-if="gameState === 'over'" 
      :score="finalScore" 
      :mode="gameMode" 
      :usedTime="usedTime" 
      :reason="endReason" 
      :highScore="highScores[gameMode]"
      @retry="retryGame" 
      @home="goHome" 
    />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import Home from './components/Home.vue';
import Game from './components/Game.vue';
import Over from './components/Over.vue';

// --- 状态机 ---
// 'home' -> 'playing' -> 'over' -> 'home' / 'playing'
const gameState = ref('home');
const gameMode = ref('classic');  // 'classic' | 'speed'
const timeLimit = ref(15);        // 手速模式倒计时秒数
const finalScore = ref(0);
const endReason = ref('wrong');   // 'wrong' | 'timeout'
const usedTime = ref(0);          // 实际消耗时间

// 最高分持久化 localStorage
const HIGH_SCORE_KEY = 'eat-the-coala-highscores';
function loadHighScores() {
  try {
    const raw = localStorage.getItem(HIGH_SCORE_KEY);
    if (raw) {
      const obj = JSON.parse(raw);
      return {
        classic: obj.classic ?? 0,
        speed: obj.speed ?? 0
      };
    }
  } catch {}
  return { classic: 0, speed: 0 };
}
const highScores = ref(loadHighScores());

// 开始游戏
const startGame = (mode = 'classic') => {
  gameMode.value = mode;
  gameState.value = 'playing';
};

// 重新开始（保持当前模式）
const retryGame = () => {
  gameState.value = 'playing';
};

// 游戏结束
const handleGameOver = ({ score, reason, time }) => {
  finalScore.value = score;
  endReason.value = reason;
  usedTime.value = time;
  updateHighScore(gameMode.value, score);

  gameState.value = 'over';
};

// 返回主页
const goHome = () => {
  gameState.value = 'home';
};

// 更新最高分
const updateHighScore = (mode, score) => {
  if (score > highScores.value[mode]) {
    highScores.value[mode] = score;
    // 保存到 localStorage
    localStorage.setItem(HIGH_SCORE_KEY, JSON.stringify(highScores.value));
  }
}

// 页面加载时同步 localStorage 数据
onMounted(() => {
  highScores.value = loadHighScores();
});
</script>

<style scoped>
.app-container {
  position: relative;
  width: 100%;
  max-width: 450px;
  height: 100vh;
  height: 100dvh;
  margin: 0 auto;
  overflow: hidden;
  user-select: none;
}
</style>
