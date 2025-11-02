<template>
  <div id="levelsPage">
    <h1>🐟 闯关模式</h1>
    <div style="margin-bottom: 16px">通过当前关卡解锁下一关</div>
    
    <div class="levels-container">
      <div
        v-for="level in levels"
        :key="level.id"
        class="level-item"
        :class="{ 'locked': !isUnlocked(level.id), 'completed': isCompleted(level.id) }"
        @click="enterLevel(level)"
      >
        <div class="level-number">{{ level.id }}</div>
        <div class="level-name">{{ level.name }}</div>
        <div class="level-difficulty">{{ level.difficulty }}</div>
        <div v-if="isCompleted(level.id)" class="completed-mark">✓</div>
        <div v-else-if="!isUnlocked(level.id)" class="locked-mark">🔒</div>
      </div>
    </div>
    
    <a-button block style="margin-top: 24px" @click="router.back()">返回主页面</a-button>
    <my-ad />
  </div>
</template>

<script setup lang="ts">
import { useRouter } from "vue-router";
import { useGlobalStore } from "../core/globalStore";
import MyAd from "../components/MyAd.vue";
import { reactive, computed } from "vue";

const router = useRouter();
const { setGameConfig } = useGlobalStore();

// 关卡配置
const levels = reactive([
  { id: 1, name: '新手村', difficulty: '简单', config: easyGameConfig },
  { id: 2, name: '小鱼儿', difficulty: '中等', config: middleGameConfig },
  { id: 3, name: '大鲨鱼', difficulty: '困难', config: hardGameConfig },
  { id: 4, name: '海洋霸主', difficulty: '地狱', config: lunaticGameConfig },
  { id: 5, name: '深渊挑战', difficulty: '天狱', config: skyGameConfig },
  { id: 6, name: '终极考验', difficulty: '羊了个羊', config: yangGameConfig },
]);

// 从缓存获取已解锁和已完成的关卡
const getUnlockedLevels = (): number[] => {
  const saved = localStorage.getItem('unlockedLevels');
  return saved ? JSON.parse(saved) : [1]; // 默认解锁第一关
};

const getCompletedLevels = (): number[] => {
  const saved = localStorage.getItem('completedLevels');
  return saved ? JSON.parse(saved) : [];
};

// 保存关卡状态到缓存
const saveUnlockedLevels = (levels: number[]) => {
  localStorage.setItem('unlockedLevels', JSON.stringify(levels));
};

const saveCompletedLevels = (levels: number[]) => {
  localStorage.setItem('completedLevels', JSON.stringify(levels));
};

// 计算属性
const unlockedLevels = computed(() => getUnlockedLevels());
const completedLevels = computed(() => getCompletedLevels());

// 检查关卡是否解锁
const isUnlocked = (levelId: number): boolean => {
  return unlockedLevels.value.includes(levelId);
};

// 检查关卡是否完成
const isCompleted = (levelId: number): boolean => {
  return completedLevels.value.includes(levelId);
};

// 进入关卡
const enterLevel = (level: any) => {
  if (isUnlocked(level.id)) {
    // 保存当前关卡ID，以便游戏结束后更新进度
    localStorage.setItem('currentLevel', JSON.stringify(level.id));
    // 通过URL参数传递关卡ID
    router.push(`/game?level=${level.id}`);
  }
};
</script>

<style scoped>
#levelsPage {
  text-align: center;
  padding: 20px;
}

.levels-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 16px;
  margin-bottom: 24px;
}

.level-item {
  width: 120px;
  height: 160px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 12px;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: white;
  cursor: pointer;
  transition: all 0.3s ease;
  position: relative;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.level-item:hover:not(.locked) {
  transform: translateY(-5px);
  box-shadow: 0 6px 12px rgba(0, 0, 0, 0.15);
}

.level-item.locked {
  background: #ccc;
  cursor: not-allowed;
  opacity: 0.6;
}

.level-item.completed {
  background: linear-gradient(135deg, #f093fb 0%, #f5576c 100%);
}

.level-number {
  font-size: 32px;
  font-weight: bold;
  margin-bottom: 8px;
}

.level-name {
  font-size: 16px;
  font-weight: 500;
  margin-bottom: 4px;
}

.level-difficulty {
  font-size: 12px;
  opacity: 0.9;
}

.completed-mark {
  position: absolute;
  top: 8px;
  right: 8px;
  font-size: 20px;
  color: #4ade80;
}

.locked-mark {
  position: absolute;
  top: 8px;
  right: 8px;
  font-size: 20px;
  color: #6b7280;
}
</style>