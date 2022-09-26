<template>
	<view class="checkerboardItem">

	</view>
</template>

<script setup>
	import {
		ref,
		toRefs,
		watchEffect,
	} from "vue"

	const props = defineProps({
		item: Array
	})

	let [x, y, num] = toRefs(props.item)
	let color = ref('')
	const colors = [
		"#4e9ad4",
		"#4ed4a1",
		"#894ed4",
		"#d44e98",
		"#e95151",
		"#e9c351",
		"#6ee951"
	]
	
	const randomColor = ()=>{
		let num = Math.floor(Math.random()*7)
		return colors[num]
	}
	watchEffect(() => {
		switch (num.value) {
			case 0:
				color.value = 'white'
				break;
			case 1:
				color.value = 'red'
				break;
			case 2:
				color.value = randomColor()
				break;
		}
	})
</script>

<style lang="scss" scoped>
	.checkerboardItem {
		width: 46.7rpx;
		height: 46.7rpx;
		border: 1rpx solid #333;
		box-sizing: border-box;
		background-color: v-bind(color);
		border-radius: 6rpx;
		box-shadow: inset 0rpx 0rpx 18rpx 0rpx #e5e5e5;
	}
</style>
