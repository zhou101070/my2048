<template>
  <div id="app-container">
    <div class="main">
      <div class="score">
        <span>分数</span>
        <span>{{ score }}</span>
      </div>
      <div class="game" ref="gameRef" :style="{ width: gameWidth + 'px' }">
        <div :class="['mask', { mask_on: !state }]">
          <span class="title">{{ count > 1 ? '游戏结束' : '' }}</span>
          <a-button type="primary" class="restart" @click="restart">
            {{ count > 1 ? '重新开始' : '开始游戏' }}
          </a-button>
          <div class="custom">
            <p>自定义游戏</p>
            <a-space>
              <span style="color: white">高度:</span>
              <a-input-number v-model:value="size.height" :min="3" />
              <span style="color: white">宽度:</span>
              <a-input-number v-model:value="size.width" :min="3" />
            </a-space>
          </div>
        </div>
        <div 
          v-for="(item, index) in items" 
          :key="index"
          :class="['item', initClass(item.value)]"
        >
          {{ item.value ? item.value : '' }}
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, computed, onMounted, onUnmounted, watch } from 'vue';

// State
const size = reactive({
  width: 4,
  height: 4,
  minWidth: 3,
  minHeight: 3
});

const counts = reactive({
  left: 0,
  right: 0,
  down: 0,
  up: 0
});

const count = ref(0);
const state = ref(false);
const score = ref(0);
const overDetermine = ref(false);
const items = ref([]);
const gameRef = ref(null);

const gameWidth = computed(() => {
  return 54 * size.width;
});

watch(() => size.width, (newValue) => {
  if (newValue < size.minWidth) {
    size.width = size.minWidth;
  }
});

watch(() => size.height, (newValue) => {
  if (newValue < size.minHeight) {
    size.height = size.minHeight;
  }
});

const init = () => {
  size.width = size.width < size.minWidth ? size.minWidth : size.width;
  size.height = size.height < size.minHeight ? size.minHeight : size.height;
  for (let i = 0; i < size.width; i++) {
    for (let j = 0; j < size.height; j++) {
      items.value.push({ value: 0 });
    }
  }
  count.value++;
};

const restart = () => {
  items.value = [];
  init();
  state.value = true;
  score.value = 0;
  Object.assign(counts, { left: 0, right: 0, down: 0, up: 0 });
  rundomGenerate();
};

const getIndex = (x, y) => {
  return size.width * y + x;
};

const getAvailableIndexs = () => {
  let arr = [];
  items.value.forEach((item, index) => {
    if (!item.value) {
      arr.push(index);
    }
  });
  return arr;
};

const gameJudge = () => {
  if (!getAvailableIndexs().length) {
    for (let i = 0; i < size.height; i++) {
      let temp;
      try {
        temp = items.value[getIndex(0, i)].value;
      } catch (e) {
        console.log(getIndex(0, i), i);
      }

      for (let j = 0; j < size.width; j++) {
        const { value } = items.value[getIndex(j, i)];
        if (!value) return true;
        if (temp === value && j) {
          return true;
        } else {
          temp = value;
        }
        if (i > 0) {
          const { value: topvalue } = items.value[getIndex(j, i - 1)];
          if (value === topvalue) {
            return true;
          }
        }
      }
    }
  } else {
    return true;
  }
  return false;
};

const gameOver = () => {
  state.value = false;
};

const rundomGenerate = () => {
  let vacancys = getAvailableIndexs();
  if (!vacancys.length) return false;

  if (vacancys.length === 1 && !overDetermine.value) {
    overDetermine.value = true;
    if (!gameJudge()) {
      gameOver();
    }
  }
  let randomIndex = Math.floor(Math.random() * vacancys.length);
  items.value[vacancys[randomIndex]].value = (Math.random() * 10 + 1) > 5 ? 4 : 2;
  return true;
};

const setscore = (value) => {
  score.value += 2 * value;
};

const setItems = (index, value) => {
  items.value[index].value = value;
};

const leftOrUp = (e, direction) => {
  let { width, height } = size;
  if (direction === 'left') {
    let temp = width;
    width = height;
    height = temp;
  }
  for (let x = 0; x < width; x++) {
    for (let y = 0; y < height - 1; y++) {
      let index = direction === 'up' ? getIndex(x, y) : getIndex(y, x);
      const { value } = items.value[index];
      if (value) {
        let k = y + 1;
        while (k < height) {
          let tindex = direction === 'up' ? getIndex(x, k) : getIndex(k, x);
          const { value: tvalue } = items.value[tindex];
          if (tvalue) {
            if (tvalue === value) {
              setItems(index, 2 * value);
              setItems(tindex, 0);
              setscore(value);
              e.flag = true;
            } else {
              let mbindex = direction === 'up' ? getIndex(x, y + 1) : getIndex(y + 1, x);
              setItems(mbindex, tvalue);
              if (tindex !== mbindex) {
                setItems(tindex, 0);
                e.flag = true;
              }
            }
            break;
          }
          k++;
        }
      } else {
        let k = y + 1;
        while (k < height) {
          let tindex = direction === 'up' ? getIndex(x, k) : getIndex(k, x);
          const { value: tvalue } = items.value[tindex];
          if (tvalue) {
            setItems(index, tvalue);
            setItems(tindex, 0);
            e.flag = true;
            y--;
            break;
          }
          k++;
        }
      }
    }
  }
};

const rightOrDown = (e, direction) => {
  let { width, height } = size;
  if (direction === 'right') {
    let temp = width;
    width = height;
    height = temp;
  }
  for (let x = 0; x < width; x++) {
    for (let y = height - 1; y > 0; y--) {
      let index = direction === 'down' ? getIndex(x, y) : getIndex(y, x);
      const { value } = items.value[index];
      if (value) {
        let k = y - 1;
        while (k >= 0) {
          let tindex = direction === 'down' ? getIndex(x, k) : getIndex(k, x);
          const { value: tvalue } = items.value[tindex];
          if (tvalue) {
            if (tvalue === value) {
              setItems(index, 2 * value);
              setItems(tindex, 0);
              setscore(value);
              e.flag = true;
            } else {
              let mbindex = direction === 'down' ? getIndex(x, y - 1) : getIndex(y - 1, x);
              setItems(mbindex, tvalue);
              if (tindex !== mbindex) {
                setItems(tindex, 0);
                e.flag = true;
              }
            }
            break;
          }
          k--;
        }
      } else {
        let k = y - 1;
        while (k >= 0) {
          let tindex = direction === 'down' ? getIndex(x, k) : getIndex(k, x);
          const { value: tvalue } = items.value[tindex];
          if (tvalue) {
            setItems(index, tvalue);
            setItems(tindex, 0);
            e.flag = true;
            y++;
            break;
          }
          k--;
        }
      }
    }
  }
};

const main = (key) => {
  let eState = { flag: false };

  switch (key) {
    case 37: // left
      leftOrUp(eState, 'left');
      counts.left++;
      break;
    case 38: // up
      leftOrUp(eState, 'up');
      counts.up++;
      break;
    case 39: // right
      rightOrDown(eState, 'right');
      counts.right++;
      break;
    case 40: // down
      rightOrDown(eState, 'down');
      counts.down++;
      break;
  }
  return eState.flag;
};

const handleKeydown = (e) => {
  let key = e.keyCode || e.which;
  if (key === 13 && !state.value) {
    restart();
    return;
  }
  // Prevent default scrolling for arrow keys
  if ([37, 38, 39, 40].includes(key)) {
    e.preventDefault();
  }
  if (state.value && main(key)) {
    setTimeout(() => {
      rundomGenerate();
    }, 30);
  }
  if (overDetermine.value && !gameJudge()) {
    gameOver();
  }
};

onMounted(() => {
  document.addEventListener('keydown', handleKeydown, false);
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleKeydown);
});

init();

const initClass = (value) => {
  let comparisonTable = {
    0: 'zero',
    2: 'item_2',
    4: 'item_4',
    8: 'item_8',
    16: 'item_16',
    32: 'item_32',
    64: 'item_64',
    128: 'item_128',
    256: 'item_256',
    512: 'item_512',
    1024: 'item_1024',
    2048: 'item_2048',
    4096: 'item_4096',
    8192: 'item_8192',
    16384: 'item_16384',
    32768: 'item_32768',
    65536: 'item_65536'
  };
  return comparisonTable[value];
};
</script>

<style lang="less">
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

html,
body {
  height: 100%;
  background-color: #000000;
  font-family: Arial, Helvetica, sans-serif;
  overflow: hidden;
}

#app-container {
  height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;

  .main {
    width: fit-content;
    transform: scale(1.5);
    
    .score {
      width: 100%;
      height: 60px;
      font-size: 20px;
      font-weight: 800;
      border-radius: 8px;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      background-color: #7fcbf7;
      margin-bottom: 10px;
      color: #333;
    }
    
    .game {
      position: relative;
      display: flex;
      flex-wrap: wrap;
      padding: 5px;
      background-color: #4d4d4d;
      border-radius: 5px;
      
      .item {
        width: 50px;
        height: 50px;
        margin: 2px;
        line-height: 50px;
        text-align: center;
        border-radius: 5px;
        transition: all 0.1s;
        font-weight: bold;
        color: #776e65;
        font-size: 20px;
      }
      
      .mask {
        width: 100%;
        height: 100%;
        visibility: hidden;
        display: flex;
        flex-direction: column;
        justify-content: center;
        align-items: center;
        color: white;
        background-color: rgba(52, 49, 48, 0.9);
        position: absolute;
        margin: 0;
        top: 0;
        left: 0;
        z-index: 10;
        border-radius: 5px;
        
        .title {
          font-size: 20px;
          font-weight: 800;
          margin-bottom: 15px;
        }
        
        .custom {
          text-align: center;
          font-size: 13px;
          margin-top: 15px;
          color: white;
          
          p {
            margin-bottom: 10px;
            font-size: 16px;
            font-weight: bold;
          }
          
          .ant-space {
            color: white;
          }
        }
        
        .restart {
          margin-top: 10px;
        }
      }
      
      .mask_on {
        visibility: visible;
      }
      
      .zero { background-color: #343130; color: transparent; }
      .item_2 { background-color: #eee4da; }
      .item_4 { background-color: #ede0c8; }
      .item_8 { background-color: #f2b179; color: #f9f6f2; }
      .item_16 { background-color: #f59563; color: #f9f6f2; }
      .item_32 { background-color: #f67c5f; color: #f9f6f2; }
      .item_64 { background-color: #f65e3b; color: #f9f6f2; }
      .item_128 { background-color: #edcf72; color: #f9f6f2; font-size: 16px; }
      .item_256 { background-color: #edcc61; color: #f9f6f2; font-size: 16px; }
      .item_512 { background-color: #edc850; color: #f9f6f2; font-size: 16px; }
      .item_1024 { background-color: #edc53f; color: #f9f6f2; font-size: 12px; }
      .item_2048 { background-color: #edc22e; color: #f9f6f2; font-size: 12px; }
      .item_4096 { background-color: #1b9f99; color: #f9f6f2; font-size: 12px; }
      .item_8192 { background-color: #2981cd; color: #f9f6f2; font-size: 12px; }
      .item_16384 { background-color: #8756ff; color: #f9f6f2; font-size: 12px; }
      .item_32768 { background-color: #c13409; color: #f9f6f2; font-size: 12px; }
      .item_65536 { background-color: #00ff95; color: #f9f6f2; font-size: 12px; }
    }
  }
}
</style>
