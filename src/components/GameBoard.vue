<script setup>
import { ref, onMounted, watch } from 'vue';

// 游戏状态
const score = ref(0);
const level = ref(1);
const gameBoard = ref(null);
const ctx = ref(null);
const boardSize = 8; // 8x8的游戏板
const tileSize = 60; // 每个水果图标的大小
const fruits = ref([]);
const selectedFruit = ref(null); // 选中的水果位置
const comboCount = ref(0); // 连击计数
const targetScore = ref(300); // 当前关卡目标分数
const movesLeft = ref(20); // 剩余移动次数
const gameOver = ref(false); // 游戏是否结束
const specialItems = ref([]); // 特殊道具位置

// 音效
const backgroundMusic = ref(null);
const selectSound = ref(null);
const matchSound = ref(null);
const comboSound = ref(null);
const specialSound = ref(null);
const levelCompleteSound = ref(null);
const isMuted = ref(false); // 是否静音

// 初始化游戏板
const initGame = () => {
  if (gameBoard.value) {
    const scale = window.devicePixelRatio || 1;
    gameBoard.value.width = boardSize * tileSize * scale;
    gameBoard.value.height = boardSize * tileSize * scale;
    gameBoard.value.style.width = `${boardSize * tileSize}px`;
    gameBoard.value.style.height = `${boardSize * tileSize}px`;
    ctx.value = gameBoard.value.getContext('2d');
    ctx.value.scale(scale, scale);
    
    // 不需要在这里添加点击事件监听，因为已经在模板中通过@click绑定了
    
    createBoard();
    loadSounds();
    resetGame();
  }
};

// 加载音效
const loadSounds = () => {
  backgroundMusic.value = new Audio('/sounds/background.mp3');
  backgroundMusic.value.loop = true;
  backgroundMusic.value.volume = 0.5;
  
  selectSound.value = new Audio('/sounds/select.mp3');
  matchSound.value = new Audio('/sounds/match.mp3');
  comboSound.value = new Audio('/sounds/combo.mp3');
  specialSound.value = new Audio('/sounds/special.mp3');
  levelCompleteSound.value = new Audio('/sounds/level_complete.mp3');
};

// 播放音效
const playSound = (sound) => {
  if (!isMuted.value && sound) {
    // 确保使用正确的sound对象
    const soundToPlay = sound.value ? sound.value : sound;
    soundToPlay.currentTime = 0;
    soundToPlay.play().catch(err => console.error('音效播放失败:', err));
  }
};

// 切换音效
const toggleMute = () => {
  isMuted.value = !isMuted.value;
  if (isMuted.value) {
    if (backgroundMusic.value) backgroundMusic.value.pause();
  } else {
    if (backgroundMusic.value) backgroundMusic.value.play().catch(err => console.error('背景音乐播放失败:', err));
  }
};

// 重置游戏
const resetGame = () => {
  score.value = 0;
  comboCount.value = 0;
  movesLeft.value = 20;
  gameOver.value = false;
  targetScore.value = level.value * 300;
  createBoard();
  playSound(backgroundMusic);
};

// 创建游戏板数据
const createBoard = () => {
  fruits.value = Array(boardSize).fill().map(() =>
    Array(boardSize).fill().map(() => Math.floor(Math.random() * 6))
  );
  
  // 添加特殊道具（每10个格子中随机放置1个）
  specialItems.value = [];
  const specialCount = Math.floor((boardSize * boardSize) / 10);
  for (let i = 0; i < specialCount; i++) {
    const row = Math.floor(Math.random() * boardSize);
    const col = Math.floor(Math.random() * boardSize);
    specialItems.value.push({ row, col, type: Math.floor(Math.random() * 2) }); // 0: 炸弹, 1: 魔法棒
  }
  
  drawBoard();
};

// 绘制游戏板
const drawBoard = () => {
  if (!ctx.value) return;
  
  const itemColors = [
    '#FF69B4', // 粉红色（口红）
    '#FFD700', // 金色（香水）
    '#DDA0DD', // 梅红色（眼影）
    '#FFC0CB', // 浅粉（腮红）
    '#E6E6FA', // 淡紫（高光）
    '#FFB6C1'  // 亮粉（唇彩）
  ];

  const itemGradients = itemColors.map(color => {
    const gradient = ctx.value.createRadialGradient(
      tileSize/2, tileSize/2, 0,
      tileSize/2, tileSize/2, tileSize/2
    );
    gradient.addColorStop(0, color);
    gradient.addColorStop(1, shadeColor(color, -20));
    return gradient;
  });

  // 清空画布
  ctx.value.clearRect(0, 0, gameBoard.value.width, gameBoard.value.height);

  fruits.value.forEach((row, i) => {
    row.forEach((fruit, j) => {
      ctx.value.save();
      ctx.value.translate(j * tileSize, i * tileSize);

      // 绘制主体
      if (fruit !== -1) {
        ctx.value.fillStyle = itemGradients[fruit];
        ctx.value.beginPath();
        drawBeautyItem(fruit);
        ctx.value.fill();

        // 添加高光效果
        ctx.value.fillStyle = 'rgba(255, 255, 255, 0.3)';
        ctx.value.beginPath();
        ctx.value.ellipse(
          tileSize * 0.3,
          tileSize * 0.3,
          tileSize * 0.15,
          tileSize * 0.1,
          Math.PI / 4,
          0,
          2 * Math.PI
        );
        ctx.value.fill();
      }
      
      // 绘制特殊道具
      const specialItem = specialItems.value.find(item => item.row === i && item.col === j);
      if (specialItem) {
        if (specialItem.type === 0) { // 炸弹
          drawBomb();
        } else if (specialItem.type === 1) { // 魔法棒
          drawMagicWand();
        }
      }

      ctx.value.restore();
    });
  });
  
  // 绘制游戏状态
  drawGameStatus();
};

// 辅助函数：颜色调整
const shadeColor = (color, percent) => {
  const num = parseInt(color.replace('#', ''), 16);
  const amt = Math.round(2.55 * percent);
  const R = (num >> 16) + amt;
  const G = (num >> 8 & 0x00FF) + amt;
  const B = (num & 0x0000FF) + amt;
  return `#${(0x1000000 + (R < 255 ? R < 1 ? 0 : R : 255) * 0x10000 + (G < 255 ? G < 1 ? 0 : G : 255) * 0x100 + (B < 255 ? B < 1 ? 0 : B : 255)).toString(16).slice(1)}`;
};

// 绘制美妆元素
const drawBeautyItem = (type) => {
  switch(type) {
    case 0: // 口红
      drawLipstick();
      break;
    case 1: // 香水
      drawPerfume();
      break;
    case 2: // 眼影
      drawEyeshadow();
      break;
    case 3: // 腮红
      drawBlush();
      break;
    case 4: // 高光
      drawHighlighter();
      break;
    case 5: // 唇彩
      drawLipGloss();
      break;
  }
};

// 绘制各种美妆元素的具体函数
const drawLipstick = () => {
  ctx.value.beginPath();
  ctx.value.moveTo(tileSize * 0.3, tileSize * 0.2);
  ctx.value.lineTo(tileSize * 0.7, tileSize * 0.2);
  ctx.value.lineTo(tileSize * 0.7, tileSize * 0.8);
  ctx.value.lineTo(tileSize * 0.3, tileSize * 0.8);
  ctx.value.closePath();
};

const drawPerfume = () => {
  ctx.value.beginPath();
  ctx.value.ellipse(
    tileSize * 0.5,
    tileSize * 0.6,
    tileSize * 0.3,
    tileSize * 0.2,
    0,
    0,
    2 * Math.PI
  );
  ctx.value.moveTo(tileSize * 0.4, tileSize * 0.4);
  ctx.value.lineTo(tileSize * 0.6, tileSize * 0.4);
  ctx.value.lineTo(tileSize * 0.6, tileSize * 0.6);
  ctx.value.lineTo(tileSize * 0.4, tileSize * 0.6);
  ctx.value.closePath();
};

const drawEyeshadow = () => {
  ctx.value.beginPath();
  ctx.value.ellipse(
    tileSize * 0.5,
    tileSize * 0.5,
    tileSize * 0.35,
    tileSize * 0.25,
    0,
    0,
    2 * Math.PI
  );
};

const drawBlush = () => {
  ctx.value.beginPath();
  ctx.value.moveTo(tileSize * 0.3, tileSize * 0.3);
  ctx.value.quadraticCurveTo(
    tileSize * 0.5,
    tileSize * 0.2,
    tileSize * 0.7,
    tileSize * 0.3
  );
  ctx.value.quadraticCurveTo(
    tileSize * 0.8,
    tileSize * 0.5,
    tileSize * 0.7,
    tileSize * 0.7
  );
  ctx.value.quadraticCurveTo(
    tileSize * 0.5,
    tileSize * 0.8,
    tileSize * 0.3,
    tileSize * 0.7
  );
  ctx.value.quadraticCurveTo(
    tileSize * 0.2,
    tileSize * 0.5,
    tileSize * 0.3,
    tileSize * 0.3
  );
};

const drawHighlighter = () => {
  ctx.value.beginPath();
  const points = [
    [0.5, 0.2],
    [0.65, 0.35],
    [0.8, 0.5],
    [0.65, 0.65],
    [0.5, 0.8],
    [0.35, 0.65],
    [0.2, 0.5],
    [0.35, 0.35]
  ];
  ctx.value.moveTo(tileSize * points[0][0], tileSize * points[0][1]);
  for (let i = 1; i < points.length; i++) {
    ctx.value.lineTo(tileSize * points[i][0], tileSize * points[i][1]);
  }
  ctx.value.closePath();
};

const drawLipGloss = () => {
  ctx.value.beginPath();
  ctx.value.moveTo(tileSize * 0.3, tileSize * 0.5);
  ctx.value.bezierCurveTo(
    tileSize * 0.3,
    tileSize * 0.3,
    tileSize * 0.7,
    tileSize * 0.3,
    tileSize * 0.7,
    tileSize * 0.5
  );
  ctx.value.bezierCurveTo(
    tileSize * 0.7,
    tileSize * 0.7,
    tileSize * 0.3,
    tileSize * 0.7,
    tileSize * 0.3,
    tileSize * 0.5
  );
  ctx.value.closePath();

};

// 绘制炸弹
const drawBomb = () => {
  // 绘制炸弹主体
  ctx.value.fillStyle = '#333';
  ctx.value.beginPath();
  ctx.value.arc(tileSize * 0.5, tileSize * 0.5, tileSize * 0.3, 0, 2 * Math.PI);
  ctx.value.fill();
  
  // 绘制导火线
  ctx.value.strokeStyle = '#FFA500';
  ctx.value.lineWidth = 3;
  ctx.value.beginPath();
  ctx.value.moveTo(tileSize * 0.5, tileSize * 0.2);
  ctx.value.quadraticCurveTo(tileSize * 0.7, tileSize * 0.1, tileSize * 0.8, tileSize * 0.3);
  ctx.value.stroke();
};

// 绘制魔法棒
const drawMagicWand = () => {
  // 绘制魔法棒主体
  ctx.value.fillStyle = '#9370DB';
  ctx.value.beginPath();
  ctx.value.moveTo(tileSize * 0.3, tileSize * 0.7);
  ctx.value.lineTo(tileSize * 0.7, tileSize * 0.3);
  ctx.value.lineTo(tileSize * 0.75, tileSize * 0.35);
  ctx.value.lineTo(tileSize * 0.35, tileSize * 0.75);
  ctx.value.closePath();
  ctx.value.fill();
  
  // 绘制星星
  ctx.value.fillStyle = '#FFD700';
  const starPoints = 5;
  const outerRadius = tileSize * 0.15;
  const innerRadius = tileSize * 0.07;
  ctx.value.save(); // 保存当前坐标系状态
  ctx.value.beginPath();
  ctx.value.translate(tileSize * 0.25, tileSize * 0.25);
  
  for (let i = 0; i < starPoints * 2; i++) {
    const radius = i % 2 === 0 ? outerRadius : innerRadius;
    const angle = (i * Math.PI) / starPoints;
    const x = radius * Math.cos(angle);
    const y = radius * Math.sin(angle);
    if (i === 0) {
      ctx.value.moveTo(x, y);
    } else {
      ctx.value.lineTo(x, y);
    }
  }
  ctx.value.closePath();
  ctx.value.fill();
  ctx.value.restore(); // 恢复之前保存的坐标系状态
};

// 绘制游戏状态
const drawGameStatus = () => {
  // 在画布外部显示游戏状态，这里不需要绘制，因为我们使用Vue的模板来显示
};

// 检查是否可以消除
const checkMatches = () => {
  let hasMatches = false;
  let matchCount = 0;
  
  // 检查水平方向
  for (let i = 0; i < boardSize; i++) {
    for (let j = 0; j < boardSize - 2; j++) {
      if (fruits.value[i][j] !== -1 &&
          fruits.value[i][j] === fruits.value[i][j + 1] &&
          fruits.value[i][j] === fruits.value[i][j + 2]) {
        
        // 检查是否有特殊道具
        const hasSpecial = checkSpecialItems(i, j) || 
                          checkSpecialItems(i, j + 1) || 
                          checkSpecialItems(i, j + 2);
        
        fruits.value[i][j] = -1;
        fruits.value[i][j + 1] = -1;
        fruits.value[i][j + 2] = -1;
        hasMatches = true;
        matchCount++;
        
        // 基础分数
        let points = 30;
        
        // 连击奖励
        if (comboCount.value > 0) {
          points += comboCount.value * 10;
          playSound(comboSound);
        } else {
          playSound(matchSound);
        }
        
        // 特殊道具奖励
        if (hasSpecial) {
          points *= 2;
          playSound(specialSound);
        }
        
        score.value += points;
      }
    }
  }
  
  // 检查垂直方向
  for (let i = 0; i < boardSize - 2; i++) {
    for (let j = 0; j < boardSize; j++) {
      if (fruits.value[i][j] !== -1 &&
          fruits.value[i][j] === fruits.value[i + 1][j] &&
          fruits.value[i][j] === fruits.value[i + 2][j]) {
        
        // 检查是否有特殊道具
        const hasSpecial = checkSpecialItems(i, j) || 
                          checkSpecialItems(i + 1, j) || 
                          checkSpecialItems(i + 2, j);
        
        fruits.value[i][j] = -1;
        fruits.value[i + 1][j] = -1;
        fruits.value[i + 2][j] = -1;
        hasMatches = true;
        matchCount++;
        
        // 基础分数
        let points = 30;
        
        // 连击奖励
        if (comboCount.value > 0) {
          points += comboCount.value * 10;
          playSound(comboSound);
        } else {
          playSound(matchSound);
        }
        
        // 特殊道具奖励
        if (hasSpecial) {
          points *= 2;
          playSound(specialSound);
        }
        
        score.value += points;
      }
    }
  }
  
  // 更新连击计数
  if (hasMatches) {
    comboCount.value += matchCount;
  } else {
    comboCount.value = 0;
  }
  
  // 检查是否达到目标分数
  if (score.value >= targetScore.value) {
    levelUp();
  }
  
  return hasMatches;
};

// 检查特定位置是否有特殊道具
const checkSpecialItems = (row, col) => {
  const index = specialItems.value.findIndex(item => item.row === row && item.col === col);
  if (index !== -1) {
    // 如果是炸弹，消除周围的水果
    if (specialItems.value[index].type === 0) {
      explodeBomb(row, col);
    } 
    // 如果是魔法棒，消除同类型的水果
    else if (specialItems.value[index].type === 1) {
      useMagicWand(row, col);
    }
    
    // 移除已使用的特殊道具
    specialItems.value.splice(index, 1);
    return true;
  }
  return false;
};

// 炸弹效果：消除周围的水果
const explodeBomb = (row, col) => {
  for (let i = Math.max(0, row - 1); i <= Math.min(boardSize - 1, row + 1); i++) {
    for (let j = Math.max(0, col - 1); j <= Math.min(boardSize - 1, col + 1); j++) {
      if (fruits.value[i][j] !== -1) {
        fruits.value[i][j] = -1;
        score.value += 10;
      }
    }
  }
};

// 魔法棒效果：消除同类型的水果
const useMagicWand = (row, col) => {
  const fruitType = fruits.value[row][col];
  if (fruitType === -1) return;
  
  let count = 0;
  for (let i = 0; i < boardSize; i++) {
    for (let j = 0; j < boardSize; j++) {
      if (fruits.value[i][j] === fruitType) {
        fruits.value[i][j] = -1;
        count++;
      }
    }
  }
  
  score.value += count * 15;
};

// 升级
const levelUp = () => {
  level.value++;
  targetScore.value = level.value * 300;
  movesLeft.value = 20;
  playSound(levelCompleteSound);
  createBoard();
};

// 填充空缺
const fillGaps = () => {
  // 从底部向上遍历
  for (let j = 0; j < boardSize; j++) {
    let bottom = boardSize - 1;
    for (let i = boardSize - 1; i >= 0; i--) {
      if (fruits.value[i][j] !== -1) {
        if (i !== bottom) {
          fruits.value[bottom][j] = fruits.value[i][j];
          fruits.value[i][j] = -1;
        }
        bottom--;
      }
    }
    // 填充顶部空缺
    for (let i = bottom; i >= 0; i--) {
      fruits.value[i][j] = Math.floor(Math.random() * 6);
    }
  }
};

// 交换水果并检查是否可以消除
const swapFruits = (pos1, pos2) => {
  // 如果游戏已结束，不允许交换
  if (gameOver.value) return false;
  
  // 播放选择音效
  playSound(selectSound);
  
  const temp = fruits.value[pos1.row][pos1.col];
  fruits.value[pos1.row][pos1.col] = fruits.value[pos2.row][pos2.col];
  fruits.value[pos2.row][pos2.col] = temp;
  
  if (!checkMatches()) {
    // 如果没有匹配，交换回来
    const temp = fruits.value[pos1.row][pos1.col];
    fruits.value[pos1.row][pos1.col] = fruits.value[pos2.row][pos2.col];
    fruits.value[pos2.row][pos2.col] = temp;
    return false;
  }
  
  // 减少移动次数
  movesLeft.value--;
  
  // 有匹配，填充空缺
  fillGaps();
  // 继续检查是否还有可以消除的
  while (checkMatches()) {
    fillGaps();
  }
  
  // 检查游戏是否结束
  checkGameOver();
  
  return true;
};

// 检查游戏是否结束
const checkGameOver = () => {
  // 如果达到目标分数，游戏不会结束
  if (score.value >= targetScore.value) return;
  
  // 如果没有移动次数了，游戏结束
  if (movesLeft.value <= 0) {
    gameOver.value = true;
  }
};

// 监听画布点击事件
const handleClick = (event) => {
  // 如果游戏已结束，不响应点击
  if (gameOver.value) return;
  
  const rect = gameBoard.value.getBoundingClientRect();
  const scale = window.devicePixelRatio || 1;
  
  // 简化坐标计算逻辑，直接计算点击位置对应的网格坐标
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  
  const col = Math.floor(x / (tileSize));
  const row = Math.floor(y / (tileSize));
  
  if (row < 0 || row >= boardSize || col < 0 || col >= boardSize) return;
  
  // 防止重复点击同一位置
  if (selectedFruit.value && selectedFruit.value.row === row && selectedFruit.value.col === col) {
    selectedFruit.value = null;
    drawBoard();
    return;
  }
  
  if (!selectedFruit.value) {
    selectedFruit.value = { row, col };
    // 播放选择音效
    playSound(selectSound);
    // 绘制选中效果
    drawBoard();
    ctx.value.strokeStyle = '#FFD700';
    ctx.value.lineWidth = 3;
    ctx.value.strokeRect(
      col * tileSize + 2,
      row * tileSize + 2,
      tileSize - 4,
      tileSize - 4
    );
  } else {
    const oldRow = selectedFruit.value.row;
    const oldCol = selectedFruit.value.col;
    
    // 检查是否相邻
    if ((Math.abs(row - oldRow) === 1 && col === oldCol) ||
        (Math.abs(col - oldCol) === 1 && row === oldRow)) {
      // 播放选择音效
      playSound(selectSound);
      
      // 交换水果并添加动画效果
      const startTime = Date.now();
      const animationDuration = 300;
      
      const animate = () => {
        const progress = (Date.now() - startTime) / animationDuration;
        
        if (progress < 1) {
          drawBoard();
          requestAnimationFrame(animate);
        } else {
          swapFruits(selectedFruit.value, { row, col });
          selectedFruit.value = null;
          drawBoard();
        }
      };
      
      animate();
    } else {
      // 如果不相邻，取消选择并重新选择
      selectedFruit.value = { row, col };
      playSound(selectSound);
      drawBoard();
      
      // 高亮显示选中的水果
      ctx.value.strokeStyle = '#FFD700';
      ctx.value.lineWidth = 3;
      ctx.value.strokeRect(
        col * tileSize + 2,
        row * tileSize + 2,
        tileSize - 4,
        tileSize - 4
      );
    }
  }
};

onMounted(() => {
  initGame();
});
</script>

<template>
  <div class="game-container">
    <div class="game-header">
      <div class="score-box">
        分数: {{ score }} / {{ targetScore }}
      </div>
      <div class="level-box">
        关卡: {{ level }}
      </div>
      <div class="moves-box">
        剩余步数: {{ movesLeft }}
      </div>
    </div>
    
    <canvas
      ref="gameBoard"
      @click="handleClick"
      class="game-board"
    ></canvas>
    
    <div class="game-controls">
      <button class="control-btn" @click="resetGame">重新开始</button>
      <button class="control-btn" @click="toggleMute">{{ isMuted ? '开启音效' : '静音' }}</button>
    </div>
    
    <div v-if="gameOver" class="game-over">
      <div class="game-over-content">
        <h2>游戏结束</h2>
        <p>你的分数: {{ score }}</p>
        <button class="control-btn" @click="resetGame">再玩一次</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.game-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 20px;
  background: #f0f2f5;
  border-radius: 16px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.game-header {
  display: flex;
  justify-content: space-between;
  width: 100%;
  max-width: 480px;
  margin-bottom: 20px;
}

.score-box,
.level-box,
.moves-box {
  background: white;
  padding: 10px 20px;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.05);
  font-size: 1.2em;
  font-weight: bold;
  color: #333;
}

.game-board {
  background: white;
  border-radius: 12px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  margin: 10px 0;
}

.game-controls {
  display: flex;
  gap: 20px;
  margin-top: 20px;
}

.control-btn {
  background: #4CAF50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 8px;
  font-size: 1em;
  cursor: pointer;
  transition: background-color 0.3s;
}

.control-btn:hover {
  background: #45a049;
}

.game-over {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 10;
}

.game-over-content {
  background: white;
  padding: 30px;
  border-radius: 12px;
  text-align: center;
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
}

.game-over h2 {
  color: #e91e63;
  font-size: 2em;
  margin-bottom: 20px;
}

.game-over p {
  font-size: 1.5em;
  margin-bottom: 30px;
}
</style>