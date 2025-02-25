<template>
  <div class="lottery-container">
    <div class="lottery-content">
      <h1>幸运抽奖</h1>
      
      <!-- 编辑模式切换 -->
      <div class="mode-switch">
        <button @click="isEditMode = !isEditMode" class="mode-button">
          {{ isEditMode ? '完成编辑' : '编辑奖项' }}
        </button>
      </div>

      <!-- 奖项编辑面板 -->
      <div v-if="isEditMode" class="edit-panel">
        <div v-for="(item, index) in prizes" :key="index" class="prize-item">
          <div class="prize-input-group">
            <input 
              type="text" 
              v-model="item.name"
              :placeholder="'奖项名称'"
              class="prize-input"
            >
            <input 
              type="number" 
              v-model="item.probability"
              :placeholder="'概率(1-100)'"
              class="probability-input"
              min="0"
              max="100"
            >
            <div class="prize-actions">
              <button @click="removePrize(index)" class="action-button delete">
                删除
              </button>
            </div>
          </div>
        </div>
        <button @click="addPrize" class="add-button">
          添加奖项
        </button>
        <div class="total-probability">
          总概率: {{ totalProbability }}%
          <span v-if="totalProbability !== 100" class="warning">
            (请调整至100%)
          </span>
        </div>
      </div>

      <!-- 抽奖区域 -->
      <div class="cards-container" :class="{ 'no-clicking': isSpinning }">
        <div 
          v-for="(card, index) in cards" 
          :key="index"
          class="card"
          :class="{ 'is-flipped': card.isFlipped }"
          @click="flipCard(index)"
        >
          <div class="card-face card-front">
            <span class="card-number">{{ index + 1 }}</span>
          </div>
          <div class="card-face card-back">
            <div class="card-content" :data-prize="card.prize">
              <div class="prize-name">{{ card.prize }}</div>
              <div class="prize-icon">
                {{ getPrizeIcon(card.prize) }}
              </div>
            </div>
          </div>
        </div>
      </div>

      <!-- 重置按钮 -->
      <button 
        @click="resetCards" 
        :disabled="isSpinning || !hasFlippedCard"
        class="reset-button"
      >
        重新开始
      </button>

      <!-- 中奖提示 -->
      <transition name="fade">
        <div v-if="currentWinner" class="winner" :class="{ 'no-prize': currentWinner === '谢谢参与' }">
          <div class="firework"></div>
          <div class="firework"></div>
          <div class="firework"></div>
          <div class="winner-content">
            <div class="shine"></div>
            <h2>
              <template v-if="currentWinner === '谢谢参与'">
                很遗憾，下次再来试试吧 👋
              </template>
              <template v-else>
                🎉 恭喜获得 {{ currentWinner }} 🎉
              </template>
            </h2>
          </div>
        </div>
      </transition>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, computed } from 'vue'

interface Prize {
  name: string
  probability: number
}

interface Card {
  isFlipped: boolean
  prize: string
}

// const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEEAD', '#D4A5A5']
const isEditMode = ref(false)
const isSpinning = ref(false)
const currentWinner = ref<string | null>(null)
const prizes = ref<Prize[]>([
  { name: '一等奖', probability: 5 },
  { name: '二等奖', probability: 15 },
  { name: '三等奖', probability: 30 },
  { name: '谢谢参与', probability: 50 }
])

// 生成卡片数组
const generateCards = () => {
  const cards: Card[] = []
  const totalCards = 12 // 固定12张卡片
  
  // 根据概率分配奖项
  const prizePool: string[] = []
  prizes.value.forEach(prize => {
    const count = Math.round((prize.probability / 100) * totalCards)
    for (let i = 0; i < count; i++) {
      prizePool.push(prize.name)
    }
  })
  
  // 如果奖池数量不足，补充"谢谢参与"
  while (prizePool.length < totalCards) {
    prizePool.push('谢谢参与')
  }
  
  // 打乱奖池
  for (let i = prizePool.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1))
    ;[prizePool[i], prizePool[j]] = [prizePool[j], prizePool[i]]
  }
  
  // 生成卡片
  for (let i = 0; i < totalCards; i++) {
    cards.push({
      isFlipped: false,
      prize: prizePool[i]
    })
  }
  
  return cards
}

const cards = ref<Card[]>(generateCards())

// 计算是否有翻开的卡片
const hasFlippedCard = computed(() => {
  return cards.value.some(card => card.isFlipped)
})

// 计算总概率
const totalProbability = computed(() => {
  return prizes.value.reduce((sum, prize) => sum + Number(prize.probability), 0)
})

// 添加奖项
const addPrize = () => {
  prizes.value.push({
    name: `奖项 ${prizes.value.length + 1}`,
    probability: 0
  })
}

// 删除奖项
const removePrize = (index: number) => {
  prizes.value.splice(index, 1)
}

// 翻牌
const flipCard = async (index: number) => {
  if (isSpinning.value || cards.value[index].isFlipped) return
  
  isSpinning.value = true
  currentWinner.value = null
  
  // 翻牌动画
  cards.value[index].isFlipped = true
  
  // 显示中奖信息
  setTimeout(() => {
    currentWinner.value = cards.value[index].prize
    isSpinning.value = false
  }, 500)
}

// 重置卡片
const resetCards = () => {
  isSpinning.value = true
  currentWinner.value = null
  
  // 重置所有卡片
  setTimeout(() => {
    cards.value = generateCards()
    isSpinning.value = false
  }, 300)
}

// 添加获取奖项图标的函数
const getPrizeIcon = (prize: string) => {
  switch (prize) {
    case '一等奖':
      return '🏆'
    case '二等奖':
      return '🥈'
    case '三等奖':
      return '🥉'
    case '谢谢参与':
      return '👋'
    default:
      return '🎁'
  }
}
</script>

<style scoped>
/* 移动端优先的响应式设计 */
.lottery-container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  background: linear-gradient(135deg, #f6f8fd 0%, #e9edf7 100%);
  padding: 15px;
}

.lottery-content {
  text-align: center;
  background: rgba(255, 255, 255, 0.95);
  padding: 20px;
  border-radius: 30px;
  box-shadow: 0 20px 60px rgba(0, 0, 0, 0.1),
              0 10px 30px rgba(78, 205, 196, 0.1);
  width: 100%;
  max-width: 800px;
  backdrop-filter: blur(10px);
}

h1 {
  color: #2c3e50;
  font-size: 2em;
  margin-bottom: 30px;
  font-weight: 800;
  background: linear-gradient(135deg, #2c3e50, #3498db);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}

/* 模式切换按钮 */
.mode-switch {
  margin-bottom: 20px;
}

.mode-button {
  background: linear-gradient(135deg, #667eea, #764ba2);
  border: none;
  padding: 12px 30px;
  border-radius: 25px;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 600;
  letter-spacing: 1px;
  box-shadow: 0 5px 15px rgba(102, 126, 234, 0.4);
}

.mode-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(102, 126, 234, 0.6);
}

/* 编辑面板 */
.edit-panel {
  margin-bottom: 30px;
  padding: 25px;
  background: rgba(248, 249, 250, 0.9);
  border-radius: 20px;
  box-shadow: inset 0 2px 10px rgba(0, 0, 0, 0.05);
}

.prize-item {
  margin-bottom: 15px;
}

.prize-input-group {
  display: flex;
  gap: 15px;
  align-items: center;
  margin-bottom: 15px;
}

.prize-input, .probability-input {
  padding: 12px 15px;
  border: 2px solid #e9ecef;
  border-radius: 12px;
  font-size: 14px;
  transition: all 0.3s ease;
  background: white;
}

.prize-input:focus, .probability-input:focus {
  border-color: #667eea;
  box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.2);
  outline: none;
}

.prize-actions {
  display: flex;
  gap: 5px;
}

.action-button {
  padding: 8px 15px;
  border-radius: 10px;
  border: none;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.3s ease;
}

.action-button.delete {
  background: linear-gradient(135deg, #ff6b6b, #ee5253);
  color: white;
  padding: 8px 15px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(255, 107, 107, 0.3);
}

.action-button.delete:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 15px rgba(255, 107, 107, 0.4);
}

.add-button {
  width: 100%;
  padding: 12px;
  margin-top: 15px;
  background: linear-gradient(135deg, #4ECDC4, #45B7D1);
  color: white;
  border: none;
  border-radius: 12px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 600;
  letter-spacing: 1px;
  box-shadow: 0 5px 15px rgba(78, 205, 196, 0.3);
}

.add-button:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 20px rgba(78, 205, 196, 0.4);
}

.total-probability {
  margin-top: 15px;
  font-size: 14px;
  color: #666;
}

.warning {
  color: #ff6b6b;
  font-size: 12px;
}

/* 卡片容器 */
.cards-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(100px, 1fr));
  gap: 15px;
  padding: 20px;
  margin: 20px 0;
}

.cards-container.no-clicking {
  pointer-events: none;
}

/* 卡片样式 */
.card {
  position: relative;
  height: 180px;
  perspective: 1500px;
  cursor: pointer;
  transition: transform 0.3s ease;
}

.card:hover {
  transform: translateY(-5px);
}

.card-face {
  position: absolute;
  width: 100%;
  height: 100%;
  backface-visibility: hidden;
  transition: transform 0.8s cubic-bezier(0.4, 0, 0.2, 1);
  border-radius: 20px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-weight: bold;
  box-shadow: 0 10px 30px rgba(0, 0, 0, 0.15);
  overflow: hidden;
}

.card-front {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  font-size: 28px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.2);
}

.card-front::before {
  content: '';
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background: linear-gradient(
    45deg,
    transparent,
    rgba(255, 255, 255, 0.1),
    transparent
  );
  transform: rotate(45deg);
  animation: shimmer 3s infinite;
}

.card-back {
  background: linear-gradient(135deg, #fff 0%, #f8f9fa 100%);
  transform: rotateY(180deg);
  border: none;
  color: #2c3e50;
  /* padding: 20px; */
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  position: relative;
  overflow: hidden;
}

.card-back::before {
  content: '';
  position: absolute;
  inset: 0;
  background: linear-gradient(45deg, rgba(102, 126, 234, 0.05), rgba(118, 75, 162, 0.05));
  border-radius: 20px;
  border: 2px solid rgba(102, 126, 234, 0.15);
  box-shadow: inset 0 0 15px rgba(102, 126, 234, 0.1);
}

.card-content {
  position: relative;
  z-index: 1;
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.prize-name {
  font-size: 18px;
  font-weight: 600;
  text-align: center;
  background: linear-gradient(135deg, #2c3e50, #3498db);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  margin: 0;
  padding: 0 10px;
  line-height: 1.4;
}

.prize-icon {
  font-size: 36px;
  margin-top: 5px;
  filter: drop-shadow(0 2px 4px rgba(0, 0, 0, 0.1));
  animation: floatIcon 2s ease-in-out infinite;
}

@keyframes floatIcon {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-5px);
  }
}

/* 为不同奖项设置不同的样式 */
.card-content[data-prize="一等奖"] {
  background: linear-gradient(135deg, rgba(255, 215, 0, 0.1), rgba(255, 223, 0, 0.05));
}

.card-content[data-prize="二等奖"] {
  background: linear-gradient(135deg, rgba(192, 192, 192, 0.1), rgba(208, 208, 208, 0.05));
}

.card-content[data-prize="三等奖"] {
  background: linear-gradient(135deg, rgba(205, 127, 50, 0.1), rgba(184, 115, 51, 0.05));
}

.card.is-flipped .card-front {
  transform: rotateY(180deg);
}

.card.is-flipped .card-back {
  transform: rotateY(0);
}

/* 重置按钮 */
.reset-button {
  margin-top: 30px;
  padding: 15px 40px;
  font-size: 16px;
  color: white;
  background: linear-gradient(135deg, #4ECDC4, #45B7D1);
  border: none;
  border-radius: 30px;
  cursor: pointer;
  transition: all 0.3s ease;
  font-weight: 600;
  letter-spacing: 1px;
  box-shadow: 0 10px 20px rgba(78, 205, 196, 0.3);
}

.reset-button:disabled {
  background: linear-gradient(135deg, #ccc, #999);
  cursor: not-allowed;
  box-shadow: none;
}

.reset-button:hover:not(:disabled) {
  transform: translateY(-3px);
  box-shadow: 0 15px 30px rgba(78, 205, 196, 0.4);
}

/* 中奖提示 */
.winner {
  margin-top: 30px;
  padding: 25px;
  color: #2c3e50;
  border-radius: 20px;
  background: rgba(255, 255, 255, 0.95);
  position: relative;
  overflow: hidden;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(10px);
}

.winner.no-prize {
  background: rgba(248, 249, 250, 0.95);
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.05);
}

.winner.no-prize .firework {
  display: none;
}

.winner.no-prize .shine {
  display: none;
}

.winner.no-prize h2 {
  background: linear-gradient(135deg, #666, #999);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  animation: none;
}

.winner-content {
  position: relative;
  z-index: 1;
}

.winner h2 {
  font-size: 24px;
  margin: 0;
  animation: bounce 1.2s ease infinite;
  background: linear-gradient(135deg, #2c3e50, #3498db);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
}

/* 闪光效果 */
.shine {
  position: absolute;
  top: 0;
  left: -150%;
  width: 50%;
  height: 100%;
  background: linear-gradient(
    90deg,
    rgba(255, 255, 255, 0) 0%,
    rgba(255, 255, 255, 0.8) 50%,
    rgba(255, 255, 255, 0) 100%
  );
  animation: shine 2s infinite;
  transform: skewX(-20deg);
}

/* 礼花效果 */
.firework {
  position: absolute;
  width: 10px;
  height: 10px;
  border-radius: 50%;
}

.firework:nth-child(1) {
  background: #FF6B6B;
  animation: firework1 1.5s ease infinite;
}

.firework:nth-child(2) {
  background: #4ECDC4;
  animation: firework2 1.5s ease infinite;
}

.firework:nth-child(3) {
  background: #45B7D1;
  animation: firework3 1.5s ease infinite;
}

@keyframes shimmer {
  0% {
    transform: translate(-50%, -50%) rotate(45deg);
  }
  100% {
    transform: translate(50%, 50%) rotate(45deg);
  }
}

@keyframes bounce {
  0%, 100% {
    transform: translateY(0) scale(1);
  }
  50% {
    transform: translateY(-15px) scale(1.05);
  }
}

@keyframes shine {
  0% {
    left: -150%;
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    left: 150%;
    opacity: 0;
  }
}

@keyframes firework1 {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translate(-100px, -100px) scale(0);
    opacity: 0;
  }
}

@keyframes firework2 {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translate(100px, -100px) scale(0);
    opacity: 0;
  }
}

@keyframes firework3 {
  0% {
    transform: translate(0, 0) scale(1);
    opacity: 1;
  }
  100% {
    transform: translate(0, -140px) scale(0);
    opacity: 0;
  }
}

/* 动画 */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s ease;
}

.fade-enter-from, .fade-leave-to {
  opacity: 0;
}

/* 响应式设计 */
@media (min-width: 768px) {
  .lottery-content {
    padding: 30px;
  }

  .cards-container {
    grid-template-columns: repeat(4, 1fr);
    gap: 20px;
  }

  .card {
    height: 200px;
  }

  .card-content {
    font-size: 18px;
  }
  
  .card-content::after {
    font-size: 36px;
  }
}

@media (min-width: 1024px) {
  .lottery-content {
    padding: 40px;
    max-width: 1000px;
  }

  .cards-container {
    grid-template-columns: repeat(6, 1fr);
    gap: 25px;
  }

  h1 {
    font-size: 2em;
  }

  .card {
    height: 220px;
  }

  .card-content {
    font-size: 20px;
    gap: 20px;
  }
  
  .card-content::after {
    font-size: 40px;
  }
}

/* 横屏适配 */
@media (max-height: 600px) and (orientation: landscape) {
  .lottery-container {
    padding: 10px;
  }

  .lottery-content {
    padding: 15px;
  }

  .cards-container {
    grid-template-columns: repeat(3, 1fr);
    gap: 10px;
  }

  h1 {
    font-size: 1.2em;
    margin-bottom: 10px;
  }

  .edit-panel {
    margin-bottom: 10px;
  }

  .reset-button {
    margin-top: 10px;
  }

  .winner {
    margin-top: 10px;
  }

  .card {
    height: 160px;
  }

  .card-content {
    font-size: 16px;
    gap: 10px;
  }
  
  .card-content::after {
    font-size: 28px;
  }
}
</style>
