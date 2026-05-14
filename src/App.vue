<script setup lang="ts">
import { ref, onMounted, onUnmounted, watch, nextTick } from 'vue';

// --- 型定義 ---
type EnemyState = 'Idle' | 'Preparing';

interface Fighter {
  hp: number;
  maxHp: number;
  bleedStack: number;
}

// --- 状態管理 (Reactive State) ---
const p = ref<Fighter>({ hp: 100, maxHp: 100, bleedStack: 0 });
const e = ref<Fighter>({ hp: 150, maxHp: 150, bleedStack: 0 });
const stamina = ref(0);
const enemyState = ref<EnemyState>('Idle');
const logs = ref<string[]>(["状況：対峙中。"]);
const gameOver = ref(false);
const logContainer = ref<HTMLElement | null>(null);

// --- ユーティリティ ---
const pushLog = (msg: string) => {
  const time = new Date().toLocaleTimeString();
  logs.value.push(`[${time}] ${msg}`);
};

// ログ更新時に自動スクロール
watch(logs, async () => {
  await nextTick();
  if (logContainer.value) {
    logContainer.value.scrollTop = logContainer.value.scrollHeight;
  }
}, { deep: true });

// --- アクション ---
const handleThrust = () => {
  if (gameOver.value || enemyState.value === 'Preparing') return;
  const dmg = Math.floor(Math.random() * 15) + 35;
  e.value.hp = Math.max(0, e.value.hp - dmg);
  stamina.value = Math.min(100, stamina.value + 10);
  pushLog(`「突けぇ！」鋭い刺突！ (${dmg}ダメージ)`);
};

const handleDefense = () => {
  if (enemyState.value === 'Preparing') {
    enemyState.value = 'Idle';
    e.value.hp = Math.max(0, e.value.hp - 40);
    stamina.value = Math.min(100, stamina.value + 25);
    pushLog("「捕った！」攻撃を捌いて反撃！");
  }
};

const executeUltimate = () => {
  if (stamina.value >= 100) {
    pushLog("「状況終了！！」");
    e.value.hp = 0;
  }
};

const executeEnemyStrike = () => {
  if (enemyState.value === 'Preparing' && !gameOver.value) {
    p.value.hp = Math.max(0, p.value.hp - 25);
    p.value.bleedStack += 1;
    enemyState.value = 'Idle';
    pushLog("痛恨！出血が始まった！");
  }
};

// --- ゲームループ (敵の思考) ---
let aiInterval: number;
onMounted(() => {
  aiInterval = window.setInterval(() => {
    if (gameOver.value) return;

    if (enemyState.value === 'Idle') {
      if (Math.random() < 0.7) {
        enemyState.value = 'Preparing';
        pushLog("！警告：相手がナイフを振りかざした！");
        setTimeout(executeEnemyStrike, 1200);
      } else {
        pushLog("相手は距離を取って構え直した。");
      }
    }
  }, 3000);
});

onUnmounted(() => clearInterval(aiInterval));

// 勝敗判定
watch([() => p.value.hp, () => e.value.hp], ([newPHp, newEHp]) => {
  if ((newPHp <= 0 || newEHp <= 0) && !gameOver.value) {
    gameOver.value = true;
    pushLog(newEHp <= 0 ? "目標を制圧。状況終了。" : "行動不能。ミッション失敗。");
  }
});
</script>

<template>
  <div 
    class="min-h-screen flex flex-col items-center justify-center p-4 transition-colors duration-500 font-mono text-white"
    :class="enemyState === 'Preparing' ? 'bg-red-950' : 'bg-stone-900'"
  >
    <div class="w-full max-w-md bg-black/40 p-6 rounded-xl border border-white/20 shadow-2xl">
      <h1 class="text-xl font-bold mb-6 text-center border-b border-white/20 pb-2 text-stone-300">
        JGSDF CQC SIMULATOR
      </h1>

      <!-- 目標 HP -->
      <div class="mb-4">
        <div class="flex justify-between mb-1 text-xs uppercase tracking-widest text-stone-400">
          <span>Target HP</span>
          <span>{{ e.hp }} / {{ e.maxHp }}</span>
        </div>
        <div class="w-full bg-stone-800 h-3 rounded-full overflow-hidden border border-stone-700">
          <div 
            class="bg-red-600 h-full transition-all duration-500" 
            :style="{ width: `${(e.hp / e.maxHp) * 100}%` }"
          ></div>
        </div>
      </div>

      <!-- プレイヤー HP -->
      <div class="mb-6">
        <div class="flex justify-between mb-1 text-xs uppercase tracking-widest text-stone-400">
          <span>Self HP</span>
          <span>{{ p.hp }} / {{ p.maxHp }}</span>
        </div>
        <div class="w-full bg-stone-800 h-3 rounded-full overflow-hidden border border-stone-700">
          <div 
            class="bg-emerald-600 h-full transition-all duration-300" 
            :style="{ width: `${(p.hp / p.maxHp) * 100}%` }"
          ></div>
        </div>
        
        <div class="flex justify-between mt-3 mb-1 text-xs text-stone-400">
          <span>STAMINA (気力)</span>
          <span>{{ stamina }}%</span>
        </div>
        <div class="w-full bg-stone-800 h-1 rounded-full overflow-hidden">
          <div 
            class="bg-amber-400 h-full transition-all" 
            :style="{ width: `${stamina}%` }"
          ></div>
        </div>
      </div>

      <!-- ログ -->
      <div 
        ref="logContainer"
        class="h-44 overflow-y-auto bg-black/60 p-3 rounded mb-6 text-[10px] border border-white/5 leading-relaxed text-stone-300"
      >
        <div v-for="(log, i) in logs" :key="i" class="mb-1 border-l border-stone-700 pl-2">
          {{ log }}
        </div>
      </div>

      <!-- コントロール -->
      <div class="grid grid-cols-2 gap-3">
        <button 
          @click="handleThrust"
          :disabled="gameOver || enemyState === 'Preparing'"
          class="bg-stone-800 hover:bg-stone-700 disabled:opacity-20 py-3 rounded text-sm font-bold border border-stone-600 transition"
        >
          刺突
        </button>
        <button 
          @click="handleDefense"
          :disabled="gameOver || enemyState === 'Idle'"
          class="bg-blue-900 hover:bg-blue-800 disabled:opacity-20 py-3 rounded text-sm font-bold border border-blue-700 transition shadow-[0_0_15px_rgba(30,64,175,0.4)]"
        >
          防衛
        </button>
      </div>

      <button 
        @click="executeUltimate"
        :disabled="gameOver || stamina < 100"
        class="w-full mt-3 bg-amber-600 hover:bg-amber-500 disabled:opacity-10 py-3 rounded font-black transition text-black uppercase tracking-tighter"
      >
        【必殺・制圧突き】
      </button>

      <div v-if="gameOver" class="mt-6 text-center">
        <button @click="() => window.location.reload()" class="text-xs text-amber-500 underline uppercase tracking-widest">
          Re-engage (再起動)
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 必要に応じてカスタムスタイルを追加 */
::-webkit-scrollbar {
  width: 4px;
}
::-webkit-scrollbar-thumb {
  background: #444;
  border-radius: 2px;
}
</style>