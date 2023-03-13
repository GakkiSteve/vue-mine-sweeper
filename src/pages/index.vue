<script setup lang="ts">
interface BlockState {
  x: number
  y: number
  revealed?: boolean
  mine?: boolean
  flagged?: boolean
  adjacentMine: number
}

const WIDTH = 10
const HEIGHT = 10
// create an arra contains 10 0
const state = reactive(Array.from({ length: HEIGHT }, (_, y) => Array.from({ length: WIDTH }, (_, x): BlockState => ({ x, y, adjacentMine: 0 }))))

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

const numberColors = [
  'text-transparent',
  'text-blue-500',
  'text-green-500',
  'text-yellow-500',
  'text-orange-500',
  'text-red-500',
  'text-purple-500',
  'text-pink-500',
  'text-teal-500',

]

function updateNumbers() {
  state.forEach((raw, y) => {
    raw.forEach((block, x) => {
      if (block.mine)
        return
      getSiblings(block).forEach((b) => {
        if (b.mine)
          block.adjacentMine += 1
      })
    })
  })
}
let mineGenerated = false
const dev = false
function onRightClick(block: BlockState) {
  if (block.revealed)
    return
  block.flagged = !block.flagged
  checkGameState()
}
function onClick(e: MouseEvent, block: BlockState) {
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

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'

  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'
  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMine]
}

function expandZero(block: BlockState) {
  if (block.adjacentMine)
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
      <button
        v-for="block, x in row"
        :key="x"
        flex="~"
        items-center justify-center
        hover:bg-gray
        w-10 h-10 m="0.5"
        border="1 gray-400/10"
        :class="getBlockClass(block)"
        @click="onClick($event, block)"
        @contextmenu.prevent="onRightClick(block)"
      >
        <!-- {{ item.mine ? 'x' : item.adjacentMine }} -->
        <template v-if="block.flagged">
          <div>
            🚩
          </div>
        </template>
        <template v-else-if="block.revealed || dev">
          <div v-if="block.mine">
            💣
          </div>
          <div v-else>
            {{ block.adjacentMine }}
          </div>
        </template>
      </button>
    </div>
  </div>
</template>
