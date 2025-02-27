<script setup>
import { ref, onMounted } from 'vue';

// 游戏状态
const score = ref(0);
const level = ref(1);
const gameBoard = ref(null);
const ctx = ref(null);
const boardSize = 8; // 8x8的游戏板
const tileSize = 60; // 每个水果图标的大小
const fruits = ref([]);
const selectedFruit = ref(null); // 选中的水果位置

// 初始化游戏板
const initGame = () => {
  if (gameBoard.value) {
    gameBoard.value.width = boardSize * tileSize;
    gameBoard.value.height = boardSize * tileSize;
    ctx.value = gameBoard.value.getContext('2d');
    createBoard();
  }
};

// 创建游戏板数据
const createBoard = () => {
  fruits.value = Array(boardSize).fill().map(() =>
    Array(boardSize).fill().map(() => Math.floor(Math.random() * 6))
  );
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

      ctx.value.restore();
    });
  });
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

// 检查是否可以消除
const checkMatches = () => {
  let hasMatches = false;
  // 检查水平方向
  for (let i = 0; i < boardSize; i++) {
    for (let j = 0; j < boardSize - 2; j++) {
      if (fruits.value[i][j] !== -1 &&
          fruits.value[i][j] === fruits.value[i][j + 1] &&
          fruits.value[i][j] === fruits.value[i][j + 2]) {
        fruits.value[i][j] = -1;
        fruits.value[i][j + 1] = -1;
        fruits.value[i][j + 2] = -1;
        hasMatches = true;
        score.value += 30;
      }
    }
  }
  // 检查垂直方向
  for (let i = 0; i < boardSize - 2; i++) {
    for (let j = 0; j < boardSize; j++) {
      if (fruits.value[i][j] !== -1 &&
          fruits.value[i][j] === fruits.value[i + 1][j] &&
          fruits.value[i][j] === fruits.value[i + 2][j]) {
        fruits.value[i][j] = -1;
        fruits.value[i + 1][j] = -1;
        fruits.value[i + 2][j] = -1;
        hasMatches = true;
        score.value += 30;
      }
    }
  }
  return hasMatches;
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
  
  // 有匹配，填充空缺
  fillGaps();
  // 继续检查是否还有可以消除的
  while (checkMatches()) {
    fillGaps();
  }
  return true;
};

// 监听画布点击事件
const handleClick = (event) => {
  const rect = gameBoard.value.getBoundingClientRect();
  const x = event.clientX - rect.left;
  const y = event.clientY - rect.top;
  
  const col = Math.floor(x / tileSize);
  const row = Math.floor(y / tileSize);
  
  if (row < 0 || row >= boardSize || col < 0 || col >= boardSize) return;
  
  if (!selectedFruit.value) {
    selectedFruit.value = { row, col };
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
      swapFruits(selectedFruit.value, { row, col });
    }
    
    selectedFruit.value = null;
    drawBoard();
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
        分数: {{ score }}
      </div>
      <div class="level-box">
        关卡: {{ level }}
      </div>
    </div>
    
    <canvas
      ref="gameBoard"
      @click="handleClick"
      class="game-board"
    ></canvas>
    
    <div class="game-controls">
      <button class="control-btn">重新开始</button>
      <button class="control-btn">提示</button>
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
.level-box {
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
</style>