<template>
  <div id="jumpGamePage">
    <a-row align="space-between">
      <a-button style="margin-bottom: 8px" @click="doBack"> 返回</a-button>
      <a-button>得分：{{ score }}</a-button>
    </a-row>
    
    <div class="game-container">
      <div class="game-area" ref="gameArea">
        <div class="platform" v-for="(platform, index) in platforms" :key="index" 
          :style="{
            left: platform.x + 'px',
            bottom: platform.y + 'px',
            width: platform.width + 'px',
            height: platform.height + 'px',
            backgroundColor: platform.color
          }">
        </div>
        
        <div class="player" ref="player" 
          :style="{
            left: player.x + 'px',
            bottom: player.y + 'px',
            width: player.width + 'px',
            height: player.height + 'px',
            backgroundColor: player.color,
            transform: `rotate(${player.rotation}deg)`,
            transition: `transform ${jumpDuration}ms ease-out`
          }">
        </div>
        
        <div class="power-bar" v-if="powerBarVisible" 
          :style="{
            left: powerBar.x + 'px',
            bottom: powerBar.y + 'px',
            height: powerBar.height + 'px',
            width: powerBar.width + 'px',
            backgroundColor: '#ff4444',
            transform: `scaleY(${powerBar.power})`,
            transformOrigin: 'bottom center'
          }">
        </div>
      </div>
    </div>
    
    <div class="game-info">
      <div v-if="gameStatus === 0" class="start-screen">
        <h2>跳一跳</h2>
        <a-button type="primary" size="large" @click="startGame">开始游戏</a-button>
      </div>
      
      <div v-else-if="gameStatus === 2" class="game-over">
        <h2>游戏结束</h2>
        <p>你的得分：{{ score }}</p>
        <p>关卡：{{ currentLevel }}</p>
        <a-button type="primary" size="large" @click="restartGame">重新开始</a-button>
      </div>
      
      <div v-else-if="gameStatus === 3" class="game-win">
        <h2>恭喜你！</h2>
        <p>你完成了所有关卡！</p>
        <a-button type="primary" size="large" @click="restartGame">再玩一次</a-button>
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, onMounted, onUnmounted, computed } from 'vue';
import { useRouter } from 'vue-router';

const router = useRouter();

// 游戏状态：0 - 未开始, 1 - 进行中, 2 - 结束, 3 - 胜利
const gameStatus = ref(0);
const score = ref(0);
const currentLevel = ref(1);
const maxLevels = 5;

// 游戏区域
const gameArea = ref<HTMLElement | null>(null);
const gameAreaWidth = ref(0);
const gameAreaHeight = ref(0);

// 玩家
const player = reactive({
  x: 0,
  y: 0,
  width: 40,
  height: 40,
  color: '#ffeb3b',
  rotation: 0
});

// 平台
const platforms = reactive<Array<{
  x: number;
  y: number;
  width: number;
  height: number;
  color: string;
}>>([]);

// 跳跃相关
const isJumping = ref(false);
const jumpPower = ref(0);
const maxJumpPower = 1.5;
const powerBarVisible = ref(false);
const powerBar = reactive({
  x: 0,
  y: 0,
  width: 20,
  height: 100,
  power: 0
});
const jumpDuration = ref(0);
let powerInterval: number | null = null;
let powerDirection = 1;

// 关卡数据
const levels = [
  {
    platforms: [
      { x: 100, y: 0, width: 100, height: 20, color: '#4caf50' },
      { x: 300, y: 100, width: 100, height: 20, color: '#2196f3' },
      { x: 500, y: 50, width: 100, height: 20, color: '#ff9800' },
      { x: 700, y: 150, width: 100, height: 20, color: '#f44336' },
    ]
  },
  {
    platforms: [
      { x: 100, y: 0, width: 80, height: 20, color: '#4caf50' },
      { x: 300, y: 120, width: 80, height: 20, color: '#2196f3' },
      { x: 500, y: 60, width: 80, height: 20, color: '#ff9800' },
      { x: 700, y: 180, width: 80, height: 20, color: '#f44336' },
      { x: 200, y: 240, width: 80, height: 20, color: '#9c27b0' },
    ]
  },
  {
    platforms: [
      { x: 100, y: 0, width: 120, height: 20, color: '#4caf50' },
      { x: 400, y: 150, width: 120, height: 20, color: '#2196f3' },
      { x: 700, y: 80, width: 120, height: 20, color: '#ff9800' },
      { x: 300, y: 220, width: 120, height: 20, color: '#f44336' },
      { x: 600, y: 280, width: 120, height: 20, color: '#9c27b0' },
    ]
  },
  {
    platforms: [
      { x: 100, y: 0, width: 100, height: 20, color: '#4caf50' },
      { x: 350, y: 100, width: 90, height: 20, color: '#2196f3' },
      { x: 600, y: 50, width: 80, height: 20, color: '#ff9800' },
      { x: 200, y: 180, width: 110, height: 20, color: '#f44336' },
      { x: 500, y: 220, width: 90, height: 20, color: '#9c27b0' },
      { x: 750, y: 150, width: 80, height: 20, color: '#ffeb3b' },
    ]
  },
  {
    platforms: [
      { x: 100, y: 0, width: 100, height: 20, color: '#4caf50' },
      { x: 400, y: 120, width: 100, height: 20, color: '#2196f3' },
      { x: 700, y: 60, width: 100, height: 20, color: '#ff9800' },
      { x: 300, y: 200, width: 100, height: 20, color: '#f44336' },
      { x: 600, y: 260, width: 100, height: 20, color: '#9c27b0' },
      { x: 200, y: 320, width: 100, height: 20, color: '#ffeb3b' },
      { x: 500, y: 380, width: 100, height: 20, color: '#795548' },
    ]
  }
];

// 初始化游戏区域
const initGameArea = () => {
  if (gameArea.value) {
    gameAreaWidth.value = gameArea.value.clientWidth;
    gameAreaHeight.value = gameArea.value.clientHeight;
  }
};

// 初始化关卡
const initLevel = (levelIndex: number) => {
  if (levelIndex >= levels.length) {
    gameStatus.value = 3; // 胜利
    return;
  }
  
  const level = levels[levelIndex];
  platforms.length = 0;
  
  // 添加平台
  level.platforms.forEach(platform => {
    platforms.push({
      ...platform
    });
  });
  
  // 初始化玩家位置
  const firstPlatform = platforms[0];
  player.x = firstPlatform.x + firstPlatform.width / 2 - player.width / 2;
  player.y = firstPlatform.y + firstPlatform.height;
  player.rotation = 0;
};

// 开始游戏
const startGame = () => {
  gameStatus.value = 1;
  currentLevel.value = 1;
  score.value = 0;
  initLevel(0);
};

// 重新开始
const restartGame = () => {
  gameStatus.value = 1;
  currentLevel.value = 1;
  score.value = 0;
  initLevel(0);
};

// 返回首页
const doBack = () => {
  router.push('/');
};

// 准备跳跃
const prepareJump = () => {
  if (isJumping.value || gameStatus.value !== 1) return;
  
  powerBarVisible.value = true;
  powerBar.x = player.x + player.width / 2 - powerBar.width / 2;
  powerBar.y = player.y - powerBar.height - 10;
  powerBar.power = 0;
  powerDirection = 1;
  
  // 开始蓄力动画
  if (powerInterval) clearInterval(powerInterval);
  powerInterval = window.setInterval(() => {
    powerBar.power += 0.05 * powerDirection;
    
    if (powerBar.power >= maxJumpPower) {
      powerDirection = -1;
    } else if (powerBar.power <= 0) {
      powerDirection = 1;
    }
  }, 50);
  
  // 监听点击释放
  window.addEventListener('mouseup', performJump);
  window.addEventListener('touchend', performJump);
};

// 执行跳跃
const performJump = () => {
  if (powerInterval) {
    clearInterval(powerInterval);
    powerInterval = null;
  }
  powerBarVisible.value = false;
  
  if (isJumping.value || gameStatus.value !== 1) return;
  
  isJumping.value = true;
  const targetPlatform = platforms[1]; // 下一个平台
  
  if (!targetPlatform) {
    gameStatus.value = 2;
    isJumping.value = false;
    return;
  }
  
  // 计算跳跃距离和高度
  const dx = targetPlatform.x + targetPlatform.width / 2 - (player.x + player.width / 2);
  const dy = targetPlatform.y + targetPlatform.height - player.y;
  
  // 计算旋转角度
  const distance = Math.sqrt(dx * dx + dy * dy);
  const rotation = (dx / distance) * 360; // 根据水平距离旋转
  
  // 根据蓄力大小调整跳跃速度
  const jumpSpeed = 0.5 + powerBar.power * 0.5;
  jumpDuration.value = distance / jumpSpeed;
  
  // 执行跳跃动画
  player.rotation = rotation;
  
  // 使用动画帧实现平滑移动
  let startTime: number | null = null;
  const animateJump = (timestamp: number) => {
    if (!startTime) startTime = timestamp;
    const progress = Math.min((timestamp - startTime) / jumpDuration.value, 1);
    
    // 抛物线运动
    const x = player.x + dx * progress;
    const y = player.y + dy * progress + Math.sin(progress * Math.PI) * (distance / 3);
    
    player.x = x;
    player.y = y;
    
    if (progress < 1) {
      requestAnimationFrame(animateJump);
    } else {
      completeJump(targetPlatform);
    }
  };
  
  requestAnimationFrame(animateJump);
  
  // 移除事件监听
  window.removeEventListener('mouseup', performJump);
  window.removeEventListener('touchend', performJump);
};

// 完成跳跃
const completeJump = (targetPlatform: any) => {
  // 检查是否落在平台上
  const playerCenterX = player.x + player.width / 2;
  const playerBottomY = player.y + player.height;
  
  const platformLeft = targetPlatform.x;
  const platformRight = targetPlatform.x + targetPlatform.width;
  const platformTop = targetPlatform.y + targetPlatform.height;
  
  if (playerCenterX >= platformLeft && playerCenterX <= platformRight && 
      playerBottomY >= targetPlatform.y && playerBottomY <= platformTop) {
    // 成功落在平台上
    player.y = platformTop - player.height;
    player.rotation = 0;
    score.value += 100;
    
    // 移除第一个平台并添加新平台
    platforms.shift();
    
    // 检查是否完成当前关卡
    if (platforms.length === 0) {
      currentLevel.value++;
      if (currentLevel.value > maxLevels) {
        gameStatus.value = 3; // 胜利
      } else {
        // 进入下一关
        initLevel(currentLevel.value - 1);
      }
    } else {
      // 生成新平台
      generateNewPlatform();
    }
  } else {
    // 掉落失败
    gameStatus.value = 2;
  }
  
  isJumping.value = false;
};

// 生成新平台
const generateNewPlatform = () => {
  const lastPlatform = platforms[platforms.length - 1];
  
  // 随机生成新平台的位置和大小
  const minX = Math.max(50, lastPlatform.x - 300);
  const maxX = Math.min(gameAreaWidth.value - 50, lastPlatform.x + 300);
  const x = Math.random() * (maxX - minX) + minX;
  
  const minY = lastPlatform.y + 50;
  const maxY = lastPlatform.y + 150;
  const y = Math.random() * (maxY - minY) + minY;
  
  const width = Math.random() * 60 + 80;
  
  // 随机颜色
  const colors = ['#4caf50', '#2196f3', '#ff9800', '#f44336', '#9c27b0', '#ffeb3b'];
  const color = colors[Math.floor(Math.random() * colors.length)];
  
  platforms.push({
    x,
    y,
    width,
    height: 20,
    color
  });
};

// 游戏区域点击事件
const handleGameAreaClick = () => {
  if (gameStatus.value === 0) {
    startGame();
  } else if (gameStatus.value === 1) {
    prepareJump();
  }
};

// 监听窗口大小变化
const handleResize = () => {
  initGameArea();
};

onMounted(() => {
  initGameArea();
  window.addEventListener('resize', handleResize);
});

onUnmounted(() => {
  window.removeEventListener('resize', handleResize);
  if (powerInterval) clearInterval(powerInterval);
  window.removeEventListener('mouseup', performJump);
  window.removeEventListener('touchend', performJump);
});
</script>

<style scoped>
#jumpGamePage {
  text-align: center;
  padding: 20px;
}

.game-container {
  margin: 20px auto;
  width: 900px;
  height: 600px;
  border: 2px solid #333;
  border-radius: 8px;
  overflow: hidden;
  position: relative;
  background-color: #f0f0f0;
}

.game-area {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
  background: linear-gradient(135deg, #74b9ff 0%, #0984e3 100%);
}

.platform {
  position: absolute;
  border-radius: 4px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

.player {
  position: absolute;
  border-radius: 50%;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: all 0.3s ease;
}

.power-bar {
  position: absolute;
  border-radius: 2px;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
  transition: transform 0.1s ease;
}

.game-info {
  position: absolute;
  top: 50%;
  left: 50%;
  transform: translate(-50%, -50%);
  background-color: rgba(255, 255, 255, 0.8);
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.start-screen h2 {
  color: #2196f3;
  margin-bottom: 20px;
}

.game-over h2 {
  color: #f44336;
  margin-bottom: 10px;
}

.game-win h2 {
  color: #4caf50;
  margin-bottom: 10px;
}
</style>
