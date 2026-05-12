<script setup>
defineProps(['cell', 'pieces'])
</script>

<template>
  <div v-for="piece in pieces" :key="piece.row + '-' + piece.col">
    <div
      v-if="piece.row === cell.row && piece.col === cell.col"
      :class="['piece', piece.color, { king: piece.king }]"
    >
      <span v-if="piece.king">👑</span>
    </div>
  </div>
</template>

<style scoped>
.piece {
  width: 40px;
  height: 40px;
  border-radius: 50%;
}
.red {
  background-color: red;
}
.black {
  background-color: black;
}
.king {
  border: 3px solid gold;
}
</style>