<template>
  <div id="gamePage">
    <a-row align="space-between">
      <a-button style="margin-bottom: 8px" @click="doBack"> 返回</a-button>
      <a-button>块数：{{ clearBlockNum }} / {{ totalBlockNum }}</a-button>
    </a-row>
    <!-- 胜利 -->
    <a-row align="center">
      <div v-if="gameStatus === 3" style="text-align: center">
        <h2>恭喜，你赢啦！🎉</h2>
        <img alt="程序员鱼皮" src="../assets/kunkun.png" />
        <my-ad style="margin-top: 16px" />
      </div>
    </a-row>
    <!-- 分层选块 -->
    <a-row align="center">
      <div v-show="gameStatus > 0" class="level-board">
        <div v-for="(block, idx) in levelBlocksVal" :key="idx">
          <div
            v-if="block.status === 0"
            class="block level-block"
            :class="{
              disabled: !isHolyLight && block.lowerThanBlocks.length > 0,
            }"
            :data-id="block.id"
            :style="{
              zIndex: 100 + block.level,
              left: block.x * widthUnit + 'px',
              top: block.y * heightUnit + 'px',
            }"
            @click="() => doClickBlock(block)"
          >
            {{ block.type }}
          </div>
        </div>
      </div>
    </a-row>
    <!-- 随机选块 -->
    <a-row align="space-between" class="random-board">
      <div
        v-for="(randomBlock, index) in randomBlocksVal"
        :key="index"
        class="random-area"
      >
        <div
            v-if="randomBlock.length > 0"
            :data-id="randomBlock[0].id"
            class="block"
            @click="() => doClickBlock(randomBlock[0], index)"
          >
            {{ randomBlock[0].type }}
          </div>
        <!-- 隐藏 -->
        <div
          v-for="num in Math.max(randomBlock.length - 1, 0)"
          :key="num"
          class="block disabled"
        >
          <span v-if="canSeeRandom">
            {{ randomBlock[num].type }}
          </span>
        </div>
      </div>
    </a-row>
    <!-- 槽位 -->
    <a-row v-if="slotAreaVal.length > 0" align="center" class="slot-board">
      <div v-for="(slotBlock, index) in slotAreaVal" :key="index" class="block" :data-id="slotBlock?.id">
        {{ slotBlock?.type }}
      </div>
    </a-row>
    <!-- 技能 -->
    <div class="skill-board">
      <a-space>
        <a-button size="small" @click="doRevert">撤回</a-button>
        <a-button size="small" @click="doRemove">移出</a-button>
        <a-button size="small" @click="doShuffle">洗牌</a-button>
        <a-button size="small" @click="doBroke">破坏</a-button>
        <a-button size="small" @click="doHolyLight">圣光</a-button>
        <a-button size="small" @click="doSeeRandom">透视</a-button>
      </a-space>
    </div>
  </div>
</template>

<script setup lang="ts">
import useGame from "../core/game";
import { onMounted } from "vue";
import { useRouter } from "vue-router";
import MyAd from "../components/MyAd.vue";

const router = useRouter();

const {
  gameStatus,
  levelBlocksVal,
  randomBlocksVal,
  slotAreaVal,
  widthUnit,
  heightUnit,
  totalBlockNum,
  clearBlockNum,
  isHolyLight,
  canSeeRandom,
  doClickBlock,
  doStart,
  doShuffle,
  doBroke,
  doRemove,
  doRevert,
  doHolyLight,
  doSeeRandom,
} = useGame();

/**
 * 回上一页
 */
const doBack = () => {
  router.back();
};

onMounted(() => {
  doStart();
});
</script>

<style scoped>
.level-board {
  position: relative;
  perspective: 1000px;
}

.level-block {
  position: absolute;
  transition: all 0.3s ease;
  transform-style: preserve-3d;
}

.level-block:hover:not(.disabled) {
  transform: translateY(-5px) rotateX(5deg) rotateY(5deg);
  box-shadow: 0 10px 20px rgba(0, 0, 0, 0.1);
}

.random-board {
  margin-top: 8px;
}

.random-area {
  margin-top: 8px;
}

.slot-board {
  border: 10px solid saddlebrown;
  margin: 16px auto;
  width: fit-content;
}

.skill-board {
  text-align: center;
}

.block {
  font-size: 28px;
  width: 42px;
  height: 42px;
  line-height: 42px;
  border: 1px solid #eee;
  background: white;
  text-align: center;
  vertical-align: top;
  display: inline-block;
  position: relative;
  transform-style: preserve-3d;
  transition: all 0.3s ease;
}

.block::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  transform: rotateY(180deg);
  backface-visibility: hidden;
  border-radius: 4px;
}

.disabled {
  background: grey;
  cursor: not-allowed;
  opacity: 0.6;
}

/* 火焰消除动画 */
@keyframes burn {
  0% {
    transform: scale(1) rotateX(0deg) rotateY(0deg);
    opacity: 1;
  }
  50% {
    transform: scale(1.2) rotateX(180deg) rotateY(180deg);
    opacity: 0.8;
    background: linear-gradient(135deg, #ff6b6b 0%, #ffa500 100%);
  }
  100% {
    transform: scale(0) rotateX(360deg) rotateY(360deg);
    opacity: 0;
  }
}

.burn-animation {
  animation: burn 0.5s ease-out forwards;
}
</style>
