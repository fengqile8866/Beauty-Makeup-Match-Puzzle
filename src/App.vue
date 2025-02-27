<script setup>
import GameBoard from './components/GameBoard.vue'
</script>

<template>
  <div class="app-container">
    <h1 class="game-title">爆火美妆消消乐</h1>
    <GameBoard />
  </div>
</template>

<style>
.app-container {
  display: flex;
  flex-direction: column;
  align-items: center;
  min-height: 100vh;
  padding: 20px;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
}

.game-title {
  color: #2c3e50;
  font-size: 2.5em;
  margin-bottom: 30px;
  text-shadow: 2px 2px 4px rgba(0, 0, 0, 0.1);
}
</style>
