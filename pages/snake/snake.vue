<template>
  <view class="page">
    <!-- #ifndef H5 -->
    <u-navbar title="贪吃蛇"></u-navbar>
    <!-- #endif -->
    <view class="score"> 得分: {{ score }}</view>
    <view class="main">
      <view v-for="rowItem in row">
        <view
          v-for="colItem in col"
          :class="{
            food: isFood(rowItem - 1, colItem - 1),
            snake: isSnake(rowItem - 1, colItem - 1),
          }"
          class="snake-item"
        ></view>
      </view>
    </view>

    <view class="col">
      <image src="/static/top.svg" @click="moveHandle('up')" mode=""></image>
    </view>
    <view class="col1">
      <image src="/static/left.svg" @click="moveHandle('left')" mode=""></image>

      <image
        src="/static/right.svg"
        @click="moveHandle('right')"
        mode=""
      ></image>
    </view>
    <view class="col">
      <image
        src="/static/bottom.svg"
        @click="moveHandle('down')"
        mode=""
      ></image>
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
  </view>
</template>

<script setup>
import { ref, reactive, onMounted, onUnmounted, watch } from "vue"
const row = 20
const col = 30
const moveTime = 400
const direction = ref("right")
const score = ref(0)
const fail = ref(false)
const success = ref(false)
let timer = null
const snake = reactive({
  arr: [],
})
const food = reactive({
  arr: [],
})

onMounted(() => {
  init()
})

// watch food.arr.length 为0时，游戏结束
watch(
  () => food.arr.length,
  (val) => {
    if (val === 0) {
      clearInterval(timer)
      success.value = true
    }
  }
)

const init = () => {
  snake.arr = [
    { x: 0, y: 0 },
    { x: 0, y: 1 },
    { x: 0, y: 2 },
  ]
  food.arr = []
  score.value = 0
  success.value = false
  fail.value = false
  direction.value = "right"
  clearInterval(timer)
  for (let i = 0; i <= 20; i++) {
    food.arr.push({
      x: Math.floor(Math.random() * 20),
      y: Math.floor(Math.random() * 30),
    })
  }
  timer = setInterval(() => {
    move(direction.value)
    isEat()
    if (isCrash()) {
      fail.value = true
      clearInterval(timer)
    }
  }, moveTime)
}

// 判断是否食物
const isFood = (x, y) => {
  return food.arr.some((item) => item.x === x && item.y === y)
}

// 判断是不是蛇身
const isSnake = (x, y) => {
  return snake.arr.some((item) => item.x === x && item.y === y)
}
// 设置移动方向
const moveHandle = (value) => {
  direction.value = value
}

// 上下左右 移动
const move = (direction) => {
  const head = snake.arr[0]
  let newHead = {}
  switch (direction) {
    case "left":
      newHead = { x: head.x - 1, y: head.y }
      break
    case "right":
      newHead = { x: head.x + 1, y: head.y }
      break
    case "up":
      newHead = { x: head.x, y: head.y - 1 }
      break
    case "down":
      newHead = { x: head.x, y: head.y + 1 }
      break
  }
  snake.arr.unshift(newHead)
  snake.arr.pop()
}

// 判断是否吃到食物
const isEat = () => {
  const head = snake.arr[0]
  const index = food.arr.findIndex(
    (item) => item.x === head.x && item.y === head.y
  )
  if (index !== -1) {
    food.arr.splice(index, 1)
    snake.arr.push({ x: head.x, y: head.y })
    score.value += 10
  }
}

// 判断是否撞墙
const isCrash = () => {
  const head = snake.arr[0]
  if (head.x < 0 || head.x > row - 2 || head.y < 0 || head.y > col - 2) {
    return true
  }
  return false
}
</script>

<style lang="scss" scoped>
.score {
  padding: 30rpx 74rpx;
  font-size: 40rpx;
  font-weight: bold;
  color: #4db93f;
}
.main {
  display: flex;
  margin: 20rpx 74rpx;
  margin-top: 0rpx;
  /* #ifdef H5 */
  border: 1rpx solid #8b877e;
  background-color: #f5f5f5;
  /* #endif */
}
.snake-item {
  width: 30rpx;
  height: 30rpx;
  border: 1rpx solid #8b877e;
  box-sizing: border-box;
}

.food {
  background-color: #c5ac3f;
}

.snake {
  background-color: #4db93f;
}

.col,
.col1 {
  display: flex;
  justify-content: center;

  image {
    width: 120rpx;
    height: 120rpx;
    transition: all 0.2s;
  }
  image:active {
    transform: scale(0.95);
  }
}

.col1 {
  image:last-child {
    margin-left: 120rpx;
  }
}
</style>
