<script setup lang="ts">
import {
  ref,
  computed,
  watch,
  onMounted,
  onBeforeUnmount,
  useTemplateRef,
} from "vue";
import axios from "axios";

interface Direction {
  direction: "left" | "top" | "right" | "bottom";
  key: "ArrowLeft" | "ArrowUp" | "ArrowRight" | "ArrowDown";
  move: { x: number; y: number };
}

const constants: Direction[] = [
  { direction: "left", key: "ArrowLeft", move: { x: -1, y: 0 } },
  { direction: "top", key: "ArrowUp", move: { x: 0, y: -1 } },
  { direction: "right", key: "ArrowRight", move: { x: 1, y: 0 } },
  { direction: "bottom", key: "ArrowDown", move: { x: 0, y: 1 } },
];

const props = defineProps<{
  cellSize: number;
  boardSize: number;
  speed: number;
  isPlaying: boolean;
  stop: () => void;
  addScores: (s: number) => void;
  scores: number;
  playerName: string;
  onRefreshLeaderboard: () => void;
}>();

// template ref
const board = useTemplateRef<HTMLCanvasElement>("board");
const boardContext = ref<CanvasRenderingContext2D | null>(null);

const snake = ref<{ x: number; y: number }[]>([]);
const direction = ref<Direction>(constants[0]);
const targetCell = ref<{ x: number; y: number } | null>(null);
const isGameOver = ref(false);

const boardSizePx = computed(() => props.cellSize * props.boardSize);

function getMiddleCell() {
  return Math.round(props.boardSize / 2);
}

function resetSnake() {
  snake.value = [{ x: getMiddleCell(), y: getMiddleCell() }];
  const randomDirectionIndex = Math.floor(Math.random() * 4);
  direction.value = constants[randomDirectionIndex];
  targetCell.value = null;
}

function clear() {
  boardContext.value?.clearRect(0, 0, boardSizePx.value, boardSizePx.value);
}

function drawCell({ x, y }: { x: number; y: number }, index: number) {
  if (!boardContext.value) return;

  const cellSize = props.cellSize;
  const progress = index / snake.value.length;
  const prefersDark = window.matchMedia("(prefers-color-scheme: dark)").matches;

  // Define base colors for light and dark modes
  const baseColor = prefersDark ? [132, 204, 22] : [34, 197, 94]; // lime/green
  const endColor = prefersDark ? [200, 255, 100] : [100, 255, 150]; // lighter shade for gradient

  // Calculate gradient colors
  const r = Math.round(baseColor[0] * (1 - progress) + endColor[0] * progress);
  const g = Math.round(baseColor[1] * (1 - progress) + endColor[1] * progress);
  const b = Math.round(baseColor[2] * (1 - progress) + endColor[2] * progress);

  // Create linear gradient for the cell
  const gradient = boardContext.value.createLinearGradient(
    x * cellSize,
    y * cellSize,
    (x + 1) * cellSize,
    (y + 1) * cellSize
  );
  gradient.addColorStop(0, `rgb(${r}, ${g}, ${b})`);
  gradient.addColorStop(1, `rgb(${Math.round(r * 0.8)}, ${Math.round(g * 0.8)}, ${Math.round(b * 0.8)})`);

  // Add glow effect
  boardContext.value.shadowColor = prefersDark ? "rgba(132, 204, 22, 0.5)" : "rgba(34, 197, 94, 0.5)";
  boardContext.value.shadowBlur = 10;
  boardContext.value.shadowOffsetX = 0;
  boardContext.value.shadowOffsetY = 0;

  // Draw rounded rectangle
  const radius = cellSize * 0.2; // Corner radius for rounded effect
  boardContext.value.beginPath();
  boardContext.value.moveTo(x * cellSize + radius, y * cellSize);
  boardContext.value.arcTo(
    (x + 1) * cellSize,
    y * cellSize,
    (x + 1) * cellSize,
    (y + 1) * cellSize,
    radius
  );
  boardContext.value.arcTo(
    (x + 1) * cellSize,
    (y + 1) * cellSize,
    x * cellSize,
    (y + 1) * cellSize,
    radius
  );
  boardContext.value.arcTo(
    x * cellSize,
    (y + 1) * cellSize,
    x * cellSize,
    y * cellSize,
    radius
  );
  boardContext.value.arcTo(
    x * cellSize,
    y * cellSize,
    (x + 1) * cellSize,
    y * cellSize,
    radius
  );
  boardContext.value.closePath();

  // Fill with gradient
  boardContext.value.fillStyle = gradient;
  boardContext.value.fill();

  // Add outline
  boardContext.value.strokeStyle = prefersDark ? "rgba(255, 255, 255, 0.3)" : "rgba(0, 0, 0, 0.3)";
  boardContext.value.lineWidth = 2;
  boardContext.value.stroke();

  // Reset shadow to avoid affecting other drawings
  boardContext.value.shadowBlur = 0;
}

function getMoveDelay() {
  return (2 / Number(props.speed)) * 1000;
}

function isCellOutOfBoard({ x, y }: { x: number; y: number }) {
  return x < 0 || y < 0 || x >= props.boardSize || y >= props.boardSize;
}

function amountCellsInSnake(cell: { x: number; y: number }) {
  return snake.value.filter(({ x, y }) => x === cell.x && y === cell.y).length;
}

function isTargetNewHead() {
  if (!targetCell.value) return false;
  return (
    snake.value[0].x + direction.value.move.x === targetCell.value.x &&
    snake.value[0].y + direction.value.move.y === targetCell.value.y
  );
}

function getRandomCell() {
  return {
    x: Math.floor(Math.random() * props.boardSize),
    y: Math.floor(Math.random() * props.boardSize),
  };
}

function setTargetCell() {
  if (!targetCell.value) {
    let newTarget = getRandomCell();
    while (amountCellsInSnake(newTarget) > 0) {
      newTarget = getRandomCell();
    }
    targetCell.value = newTarget;
  }

  if (!boardContext.value || !targetCell.value) return;
  boardContext.value.beginPath();
  boardContext.value.rect(
    targetCell.value.x * props.cellSize,
    targetCell.value.y * props.cellSize,
    props.cellSize,
    props.cellSize
  );
  boardContext.value.fillStyle = "red";
  boardContext.value.fill();
  boardContext.value.closePath();
}

async function handleGameOver() {
  props.stop();
  isGameOver.value = true;

  try {
    await axios.post("https://snake-game-backend-production.up.railway.app/submit-score", {
      score: props.scores,
      name: !/\S/.test(props.playerName) ? 'Anonymous' : props.playerName
    });
    props.onRefreshLeaderboard();
  } catch (err) {
    console.error("❌ Lỗi gửi điểm:", err);
  }
}

function move() {
  if (!props.isPlaying) return;

  clear();
  setTargetCell();

  const newHeadCell = {
    x: snake.value[0].x + direction.value.move.x,
    y: snake.value[0].y + direction.value.move.y,
  };

  if (
    isCellOutOfBoard(newHeadCell) ||
    amountCellsInSnake(newHeadCell) > 0
  ) {
    handleGameOver();
    return;
  }

  if (isTargetNewHead()) {
    if (targetCell.value) {
      snake.value.unshift(targetCell.value);
      targetCell.value = null;
    }
    props.addScores(props.speed);
  } else {
    snake.value.unshift(newHeadCell);
    snake.value.pop();
  }

  if (!boardContext.value) return;
  boardContext.value.beginPath();
  snake.value.forEach((cell, idx) => drawCell(cell, idx));
  boardContext.value.closePath();

  setTimeout(move, getMoveDelay());
}

function onKeyPress(event: KeyboardEvent) {
  const newDirection = constants.find((c) => c.key === event.key);
  if (!newDirection) return;

  if (
    (direction.value.key === "ArrowLeft" && newDirection.key === "ArrowRight") ||
    (direction.value.key === "ArrowRight" && newDirection.key === "ArrowLeft") ||
    (direction.value.key === "ArrowUp" && newDirection.key === "ArrowDown") ||
    (direction.value.key === "ArrowDown" && newDirection.key === "ArrowUp")
  ) {
    return;
  }

  direction.value = newDirection;
}

watch(
  () => props.isPlaying,
  (value) => {
    clear();
    if (value) {
      isGameOver.value = false;
      resetSnake();
      move();
    }
  }
);

onMounted(() => {
  if (board.value) {
    boardContext.value = board.value.getContext("2d");
  }
  resetSnake();
  window.addEventListener("keydown", onKeyPress);
});

onBeforeUnmount(() => {
  window.removeEventListener("keydown", onKeyPress);
});
</script>

<template>
  <div class="relative flex justify-center items-center p-4">
    <canvas
      ref="board"
      class="rounded-2xl shadow-lg border-4 border-purple-500 dark:border-purple-400 bg-white dark:bg-slate-800 transition-colors"
      :width="boardSizePx"
      :height="boardSizePx"
    />

    <!-- Game Over Overlay -->
    <div
      v-if="isGameOver"
      class="absolute inset-0 flex flex-col items-center justify-center
                bg-slate-900/70 dark:bg-slate-950/80 backdrop-blur-md
                text-center rounded-2xl"
    >
      <h2 class="text-2xl font-bold text-white mb-4">Kết Thúc 🎮</h2>
      <p class="text-lg text-slate-200">
        Bạn đã đạt <span class="font-semibold">{{ props.scores }}</span> điểm
      </p>
    </div>
  </div>
</template>
