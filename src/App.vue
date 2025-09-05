<script setup lang="ts">
import SnakeCanvas from "@/components/SnakeCanvas.vue";
import SnakeValueInput from "@/components/SnakeValueInput.vue";
import { ref, onMounted } from "vue";
import axios from "axios";
import SnakeNameInput from "@/components/SnakeNameInput.vue";

document.title = "Trò Chơi Rắn";

// Game settings
const cellSize = ref(40);
const boardSize = ref(20);
const speed = ref(10);
const scores = ref(0);
const isPlaying = ref(false);
const playerName = ref('');

// Leaderboard
const leaderboard = ref<
  { id: number; score: number; time: string; name: string }[]
>([]);

async function refreshLeaderboard() {
  try {
    const res = await axios.get("https://snake-game-backend-production.up.railway.app/leaderboard");
    console.log(res.data);
    leaderboard.value = res.data;
  } catch (err) {
    console.error("❌ Lỗi lấy bảng xếp hạng:", err);
  }
}

function start() {
  isPlaying.value = true;
  scores.value = 0;
}

function stop() {
  isPlaying.value = false;
}

function addScores(score: number) {
  scores.value += score;
}

function onKeyPress(event: KeyboardEvent) {
  if (event.key === " " && !isPlaying.value) {
    start();
  }
}

onMounted(() => {
  refreshLeaderboard();
  window.addEventListener("keydown", onKeyPress);
});

</script>

<template>
  <div class="min-h-screen flex bg-slate-50 dark:bg-slate-900 transition-colors">
    <!-- Sidebar -->
    <aside
      class="w-72 bg-white dark:bg-slate-800 shadow-lg p-6 flex flex-col justify-between"
    >
      <div>
        <h1 class="text-2xl font-bold text-purple-600 dark:text-purple-400 mb-6">
          🐍 Trò Chơi Rắn
        </h1>

        <div class="space-y-4">
          <SnakeValueInput title="Tốc Độ" v-model="speed" :min="5" :max="20" />
          <SnakeNameInput title="Tên người chơi" v-model="playerName" />

          <!-- 🏆 Leaderboard -->
          <h2
            class="text-lg font-semibold text-slate-800 dark:text-slate-200 mb-2"
          >
            🏆 Bảng Xếp Hạng
          </h2>
          <div v-if="leaderboard.length" class="mt-6">
            <ul class="space-y-1 text-sm text-slate-700 dark:text-slate-300">
              <li
                v-for="item in leaderboard"
                :key="item.id"
                class="flex justify-between"
              >
                <span>Tên: {{ item.name }}</span>
                <span>Điểm: {{ item.score }}</span>
              </li>
            </ul>
          </div>
        </div>
      </div>

      <!-- Bottom: Score + Start/Stop -->
      <div class="mt-8 space-y-4">
        <div
          class="text-lg font-semibold text-slate-800 dark:text-slate-200 truncate"
        >
          Điểm: {{ scores }}
        </div>

        <button
          @click="isPlaying ? stop() : start()"
          class="w-full px-4 py-2 rounded-lg font-medium text-white bg-purple-600
                    hover:bg-purple-700 dark:bg-purple-500 dark:hover:bg-purple-600
                    shadow transition"
        >
          {{ isPlaying ? "Dừng" : "Bắt Đầu" }}
        </button>
      </div>
    </aside>

    <!-- Main Board -->
    <main class="flex-1 flex items-center justify-center p-6">
      <SnakeCanvas
        :cellSize="cellSize"
        :boardSize="boardSize"
        :speed="speed"
        :isPlaying="isPlaying"
        :scores="scores"
        :stop="stop"
        :addScores="addScores"
        :onRefreshLeaderboard="refreshLeaderboard"
        :playerName="playerName"
      />
    </main>
  </div>
</template>
