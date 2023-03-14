<script setup lang="ts">
import MineBlock from '~/components/MineBlock.vue'
import type { BlockState } from '~/types'

const WIDTH = 10
const HEIGHT = 10
// create an arra contains 10 0
const state = reactive(Array.from({ length: HEIGHT }, (_, y) => Array.from({ length: WIDTH }, (_, x): BlockState => ({ x, y, adjacentMines: 0 }))))

function generateMines(initial: BlockState) {
  for (const row of state) {
    for (const block of row) {
      if (Math.abs(initial.x - block.x) <= 1)
        continue

      if (Math.abs(initial.y - block.y) <= 1)
        continue
      block.mine = Math.random() < 0.1
    }
  }
  updateNumbers()
}

const directions = [
  [1, 1],
  [1, 0],
  [1, -1],
  [0, 1],
  [0, -1],
  [-1, -1],
  [-1, 0],
  [-1, 1],
]

function updateNumbers() {
  state.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMines += 1
      })
    })
  })
}
let mineGenerated = false
function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
  checkGameState()
}
function onClick(block: BlockState) {
  if (!mineGenerated) {
    generateMines(block)
    mineGenerated = true
  }
  block.revealed = true
  if (block.mine)
    alert('BOOM!')
  expandZero(block)
  checkGameState()
}

function expandZero(block: BlockState) {
  if (block.adjacentMines)
    return

  getSiblings(block).forEach((s) => {
    if (!s.revealed) {
      s.revealed = true
      expandZero(s)
    }
  })
}

// generateMines()

function getSiblings(block: BlockState) {
  return directions.map(([dx, dy]) => {
    const x2 = block.x + dx
    const y2 = block.y + dy
    if (x2 < 0 || x2 >= WIDTH || y2 < 0 || y2 >= HEIGHT)
      return undefined
    return state[y2][x2]
  }).filter(Boolean) as BlockState[]
}

function checkGameState() {
  const blocks = state.flat()
  if (blocks.every(block => (block.revealed || block.flagged)))
    alert('You win')
}
</script>

<template>
  minesweeper
  <!-- {{ state }} -->
  <div p-5>
    <div
      v-for="row, y in state" :key="y" flex="~"
      items-center justify-center
    >
      <MineBlock
        v-for="block, x in row"
        :key="x"
        :block="block"
        @click="onClick(block)"
        @contextmenu.prevent="onRightClick(block)"
      />
    </div>
  </div>
</template>
