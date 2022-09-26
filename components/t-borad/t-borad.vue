<template>
	<view class="borad">
		<t-borad-item v-for="(item,i) in boradInfo" :key="i+'aaa'" :item="item"></t-borad-item>
	</view>
</template>

<script setup>
	const props = defineProps({
		boradInfo:Array
	})
</script>

<style lang="scss" scoped>
.borad{
	display: flex;
	flex-wrap: wrap;
	border: 1rpx solid #333;
	box-sizing: border-box;
}
</style>