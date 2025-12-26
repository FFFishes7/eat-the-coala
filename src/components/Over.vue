<template>
  <div class="mask">
    <h2>{{ title }}</h2>
    
    <!-- 经典模式：显示分数 -->
    <template v-if="mode === 'classic'">
      <p class="label">得分</p>
      <p class="final-score">{{ score }}</p>
      <p class="high-score">最高分: {{ highScore }}</p>
    </template>
    
    <!-- 手速模式：显示 CPS -->
    <template v-else>
      <p class="label">{{ displayTime }}秒内点击</p>
      <p class="final-score">{{ score }}</p>
      <p class="high-score">最高分: {{ highScore }}</p>
      <p class="cps">CPS: {{ cps }}</p>

    </template>
    
    <button class="retry-btn" :disabled="isLocked" @click.stop="handleRetry" @touchstart.stop.prevent="handleRetry">Try Again</button>
    <button class="home-btn" :disabled="isLocked" @click.stop="handleHome" @touchstart.stop.prevent="handleHome">返回主页</button>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue';

const isLocked = ref(true);

onMounted(() => {
  setTimeout(() => {
    isLocked.value = false;
  }, 300);
});

const props = defineProps({
  score: {
    type: Number,
    default: 0
  },
  mode: {
    type: String,
    default: 'classic'
  },
  usedTime: {
    type: Number,
    default: 0
  },
  reason: {
    type: String,
    default: 'wrong'  // 'wrong' | 'timeout'
  },
  highScore: {
    type: Number,
    default: 0
  }
});

const emit = defineEmits(['retry', 'home']);

const handleRetry = () => {
  if (isLocked.value) return;
  emit('retry');
};

const handleHome = () => {
  if (isLocked.value) return;
  emit('home');
};

// 根据死因显示标题
const title = computed(() => {
  if (props.reason === 'timeout') {
    return 'TIME UP!';
  }
  return 'GAME OVER';
});

// 显示时间（保留1位小数）
const displayTime = computed(() => {
  return props.usedTime.toFixed(1);
});

// 计算 CPS（每秒点击数）
const cps = computed(() => {
  if (props.usedTime <= 0) return '0.00';
  return (props.score / props.usedTime).toFixed(2);
});
</script>

<style scoped>
.mask {
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.85);
  z-index: 100;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: #fff;
}

.label {
  font-size: 1rem;
  opacity: 0.7;
  margin-bottom: 5px;
}

.final-score {
  font-size: 4rem;
  margin: 0 0 10px 0;
  font-weight: bold;
}

.high-score {
  font-size: 1.5rem;
  font-weight: bold;
  color: #FFD700;
  margin-bottom: 10px;
  text-shadow: 0 2px 8px rgba(0,0,0,0.3);
}

.cps {
  font-size: 1.5rem;
  color: #FF9800;
  margin-bottom: 20px;
}

button {
  padding: 15px 40px;
  font-size: 1.5rem;
  border: none;
  background: #4CAF50;
  color: white;
  border-radius: 8px;
  cursor: pointer;
}

.home-btn {
  margin-top: 15px;
  background: transparent;
  border: 2px solid rgba(255, 255, 255, 0.5);
}

.home-btn:hover {
  background: rgba(255, 255, 255, 0.1);
  border-color: #fff;
}
</style>
