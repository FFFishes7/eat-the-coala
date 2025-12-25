<template>
  <div class="mask">
    <h1 class="title">🐨 吃掉小考拉</h1>
    
    <div class="key-hints">
      <div class="key-row">
        <span class="key">G</span>
        <span class="key">H</span>
        <span class="key">J</span>
        <span class="key">K</span>
      </div>
    </div>

    <div class="mode-buttons">
      <button class="mode-btn classic" :disabled="isLocked" @click="startGame('classic')" @touchstart.prevent="startGame('classic')">
        经典模式
      </button>
      <button class="mode-btn speed" :disabled="isLocked" @click="startGame('speed')" @touchstart.prevent="startGame('speed')">
        手速模式
        <span class="mode-desc">15秒挑战</span>
      </button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue';

const emit = defineEmits(['start']);

const isLocked = ref(true);

const startGame = (mode) => {
  if (isLocked.value) return;
  emit('start', mode);
};

onMounted(() => {
  setTimeout(() => {
    isLocked.value = false;
  }, 300);
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

.title {
  font-size: 2.5rem;
  margin-bottom: 40px;
}

.key-hints {
  margin-bottom: 40px;
}

.key-row {
  display: flex;
  gap: 10px;
}

.key {
  width: 50px;
  height: 50px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.3);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 1.5rem;
  font-weight: bold;
}

.mode-buttons {
  display: flex;
  flex-direction: column;
  gap: 15px;
  width: 200px;
}

.mode-btn {
  padding: 15px 40px;
  font-size: 1.3rem;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  position: relative;
}

.mode-btn.classic {
  background: #4CAF50;
  color: white;
}

.mode-btn.speed {
  background: #FF9800;
  color: white;
}

.mode-desc {
  display: block;
  font-size: 0.8rem;
  opacity: 0.8;
  margin-top: 2px;
}
</style>
