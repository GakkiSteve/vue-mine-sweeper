<script setup lang="ts">
import type { BlockState } from '~/types'
defineProps<{ block: BlockState }>()
const dev = false

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

function getBlockClass(block: BlockState) {
  if (block.flagged)
    return 'bg-gray-500/10'

  if (!block.revealed)
    return 'bg-gray-500/10 hover:bg-gray-500/20'
  return block.mine ? 'bg-red-500/50' : numberColors[block.adjacentMines]
}
</script>

<template>
  <button

    flex="~"
    items-center justify-center
    hover:bg-gray
    w-10 h-10 m="0.5"
    border="1 gray-400/10"
    :class="getBlockClass(block)"
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
        {{ block.adjacentMines }}
      </div>
    </template>
  </button>
</template>
