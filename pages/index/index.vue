<template>
  <view class="page">
    <view class="top">
      <view class="title">休闲一刻</view>
      <image src="/static/logo.svg" mode="widthFix" />
    </view>
    <view class="game">
      <view
        class="game-item"
        v-for="item in gameList"
        :key="item.name"
        @click="toGame(item)"
      >
        <image class="game-item-logo" :src="item.logo" mode="scaleToFill" />
        <view class="game-item-name">{{ item.name }}</view>
      </view>
    </view>
  </view>
</template>

<script setup>
import { ref } from "vue"
const gameList = ref([
  {
    name: "俄罗斯方块",
    path: "/pages/tetris/tetris",
    logo: "/static/logo/tetris.png",
  },
  {
    name: "麻了个麻",
    path: "/pages/ylgy/ylgy",
    logo: "/static/logo/ylgy.jpg",
  },
  {
    name: "贪吃蛇",
    path: "/pages/snake/snake",
    logo: "/static/logo/snake.png",
  },
])

const toGame = (item) => {
  uni.$u.route(item.path)
}
</script>

<style scoped lang="scss">
.page {
  min-height: 100vh;
  padding: 50rpx;
  box-sizing: border-box;
  background-image: linear-gradient(
    -10deg,
    #fff -8.8%,
    #fff 12.4%,
    #fff 35.6%,
    #fff 70%,
    #b1daff 90%,
    #deb1ff 100.9%
  );
}

.top {
  font-size: 80rpx;
  color: #000000;
  text-align: center;
  padding: 100rpx;
  letter-spacing: 30rpx;
  .title {
    margin-bottom: 50rpx;
  }
  image {
    width: 100%;
  }
}

.game {
  display: flex;
  flex-wrap: wrap;
  align-items: center;
  padding-top: 80rpx;
  &-item {
    width: 180rpx;
    height: 180rpx;
    display: flex;
    flex-direction: column;
    align-items: center;
    margin-right: 55rpx;
    margin-bottom: 45rpx;
    &:nth-child(3n) {
      margin-right: 0;
    }
    &-logo {
      width: 140rpx;
      height: 140rpx;
      border-radius: 26rpx;
      box-shadow: 6rpx 6rpx 14rpx #0000004b;
    }
    &-name {
      margin-top: 10rpx;
      font-size: 24rpx;
      color: #000000;
    }
  }
}
</style>
