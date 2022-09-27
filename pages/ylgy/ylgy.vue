<template>
  <view class="page">
    <!-- #ifndef H5 -->
    <u-navbar title="麻了个麻"></u-navbar>
    <!-- #endif -->
    <view class="score"> 得分: {{ score }}</view>
    <view class="main">
      <view
        class="block"
        v-for="(item, index) in sheepList.arr"
        :key="index"
        :style="itemStyle(item)"
        @click="addBar(item)"
      >
      </view>
    </view>

    <view class="bar">
      <view
        class="block"
        :class="{ 'block-move': item.move }"
        v-for="(item, index) in barList.arr"
        :key="index"
        :style="{ backgroundImage: `url(/static/ylgy/${item.color})` }"
        @click="addBar(item)"
      >
      </view>
    </view>
    <u-modal
      v-model="fail"
      content="游戏失败, 是否重新开始?"
      @confirm="init"
    ></u-modal>
    <u-modal
      v-model="success"
      :content="'游戏完成, 您的得分为' + score + ', 是否继续?'"
      @confirm="init"
      confirm-text="继续"
    ></u-modal>
    <view class="btn">
      <u-button type="primary" plain @click="refresh()">刷新</u-button>
      <u-button type="success" plain @click="init()">重玩</u-button>
    </view>
  </view>
</template>

<script setup>
import { onLoad } from "@dcloudio/uni-app"
import { ref, reactive, watch } from "vue"
onLoad(() => {})
// 得分
const score = ref(0)
const fail = ref(false)
const success = ref(false)
// 羊群
const sheepList = reactive({
  arr: [],
})
// 颜色
let colors = []

onLoad(() => {
  init()
})

watch(sheepList, (val) => {})

// 随机一个颜色
const randomColor = () => {
  return colors[Math.floor(Math.random() * colors.length)]
}

// 随机position top 50-1150 left 50-600
const randomPosition = () => {
  return {
    top: Math.floor(Math.random() * 900 + 50),
    left: Math.floor(Math.random() * 550 + 30),
    zIndex: Math.floor(Math.random() * 10 + 1),
  }
}

// 格子样式
const itemStyle = (item) => {
  return {
    backgroundImage: `url(/static/ylgy/${item.color})`,
    top: item.position.top + "rpx",
    left: item.position.left + "rpx",
    zIndex: item.position.zIndex,
    filter: item.isBottom ? "brightness(0.6)" : "brightness(1)",
  }
}

// 数组随机排序
const randomSort = (arr) => {
  return arr.sort(() => {
    return Math.random() - 0.5
  })
}

// 判断上层是否有障碍item.position.top item.position.left
const isOverlap = (item) => {
  let isOverlap = false
  sheepList.arr.forEach((i) => {
    if (
      item.position.top - 104 <= i.position.top &&
      item.position.top + 104 >= i.position.top &&
      item.position.left - 70 <= i.position.left &&
      item.position.left + 70 >= i.position.left &&
      item.position.zIndex < i.position.zIndex
    ) {
      isOverlap = true
      throw new Error("EndIterative")
    }
  })
  return isOverlap
}

// 判断每一个item上层是否有遮挡 如果没有遮挡则设置isBottom为true 否则为false
const isBottom = () => {
  sheepList.arr.forEach((item) => {
    item.isBottom = false
    try {
      isOverlap(item)
    } catch (e) {
      item.isBottom = true
    }
  })
}

// 初始化方法
const init = () => {
  sheepList.arr = []
  score.value = 0
  barList.arr = []
  let colorList = []
  for (let i = 1; i <= 33; i++) {
    colorList.push(i + ".png")
  }
  colors = randomSort(colorList).slice(0, 7)

  let arr = []
  for (let i = 0; i < 300; i = i + 3) {
    const color = randomColor()
    arr.push({
      id: i + 1,
      color: color,
      move: false,
      position: randomPosition(),
    })
    arr.push({
      id: i + 2,
      color: color,
      move: false,
      position: randomPosition(),
    })
    arr.push({
      id: i + 3,
      color: color,
      move: false,
      position: randomPosition(),
    })
  }
  sheepList.arr = randomSort(arr)
  isBottom()
}

// 刷新
const refresh = () => {
  let arr = sheepList.arr
  arr.forEach((item) => {
    item.position = randomPosition()
  })
  sheepList.arr = randomSort(arr)
  isBottom()
}

// 底部bar
const barList = reactive({
  arr: [],
})

// 是否正在移动
const isMove = ref(false)
// 移动羊的方法
const addBar = (item) => {
  if (barList.arr.length < 7 && !isMove.value) {
    // 判断是否被遮挡
    if (isOverlap(item)) {
      return
    }
    barList.arr.push(item)
    sheepList.arr = sheepList.arr.filter((i) => i.id !== item.id)
    if (barList.arr.length >= 3) {
      let colorList = {}
      barList.arr.forEach((item) => {
        if (colorList[item.color]) {
          colorList[item.color]++
        } else {
          colorList[item.color] = 1
        }
        Object.keys(colorList).forEach((item) => {
          if (colorList[item] === 3) {
            isMove.value = true
            score.value += 15
            barList.arr.map((i) => {
              if (i.color === item) {
                i.move = true
              }
            })
            setTimeout(() => {
              barList.arr = barList.arr.filter((i) => i.color !== item)
              isMove.value = false
              isFail()
            }, 500)
          }
        })
      })
    }
  }
  isBottom()
  isSuccess()
  isFail()
}

const isFail = () => {
  if (barList.arr.length === 7 && !isMove.value) {
    fail.value = true
  }
}

const isSuccess = () => {
  if (sheepList.arr.length === 0) {
    success.value = true
  }
}
</script>

<style lang="scss" scoped>
.page {
  padding: 30rpx;
  min-height: 100vh;
  background-color: #f5f5f5;
  box-sizing: border-box;
}

.score {
  font-weight: bold;
  font-size: 40rpx;
  color: #39c95d;
  margin-bottom: 20rpx;
}
.main {
  width: 100%;
  background-color: #39c95d;
  height: 1100rpx;
  box-sizing: border-box;
  padding: 20rpx;
  border-radius: 20rpx;
  display: flex;
  flex-wrap: wrap;
  align-content: flex-start;
  justify-content: space-around;
  position: relative;
}

.block {
  width: 70rpx;
  height: 104rpx;
  border-radius: 12rpx;
  overflow: hidden;
  margin: 0 20rpx 20rpx 0;
  position: absolute;
  border: 4rpx solid #e89028;
  box-shadow: 5rpx 5rpx 10rpx #474747b9;
  box-sizing: border-box;
  color: #fff;
  //   background-image: url("/static/ylgy/1 (1).png");
  background-size: 104%;
  background-repeat: no-repeat;
  background-position-y: -2rpx;
  background-position-x: -2rpx;

  //   transition: 0.5s;
  &-move {
    animation: move 0.6s linear;
  }
  &:nth-child(7n) {
    margin-right: 0;
  }
}

@keyframes move {
  0% {
    transform: scale(100%);
  }
  100% {
    transform: scale(0%);
  }
}
.bar {
  background-color: #a89a91;
  height: 104rpx;
  padding: 20rpx;
  margin-top: 50rpx;
  border-radius: 20rpx;
  display: flex;

  .block {
    position: relative;
    box-shadow: 0px 0px 0rem #474747;
  }
  .block:last-child {
    margin-right: 0;
  }
}

.btn {
  display: flex;
  margin-top: 30rpx;
}
</style>
