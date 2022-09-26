<template>
	<view class="game-page">
		<view>{{scoreNum}}</view>
		<t-borad :boradInfo="checkerboardInfo"></t-borad>
		<view class="col">
			<image src="/static/top.svg" @click="moveHandle(38)" mode=""></image>
		</view>
		<view class="col1">
			<image src="/static/left.svg" @click="moveHandle(37)" mode=""></image>

			<image src="/static/right.svg" @click="moveHandle(39)" mode=""></image>
		</view>
		<view class="col">
			<image src="/static/bottom.svg" @click="moveHandle(40)" mode=""></image>
		</view>
	</view>
</template>

<script setup>
	import {
		reactive,
		ref,
		watchEffect
	} from 'vue';
	import {
		onLoad
	} from "@dcloudio/uni-app"

	const moveHandle = (num) => {
		if (num == 38) {
			toggleSquareShape(squareTypeIndex);
		}
		moveSquare(num);
	}

	//游戏棋盘（作为俄罗斯方块在各个格子上渲染颜色的依据）
	let checkerboardInfo = reactive([]);

	let scoreNum = ref(0);

	//存储当前已经稳定坠落的方块的坐标
	let stabilitySquareCoordinate = [];

	//存储当前还在坠落的方格坐标
	let moveSquareCoordinate = null;

	//失败标杆
	let isFail = ref(false)

	//方块类型(长方形、三角形、正方形，z字型, 反z字型，l型， 反l型)
	let squareType = [0, 1, 2, 3, 4, 5, 6];

	//方块类型下标
	let squareTypeIndex = -1;

	//是否有方块出现
	let isShowSquare = ref(false);

	//随机选取方块
	const randomSquareType = () => {
		squareTypeIndex = Math.floor(Math.random() * (7));
	};

	//设置游戏界面
	const setCheckerboard = () => {
		for (let i = 0; i < 25; i++) {
			for (let j = 0; j < 16; j++) {
				checkerboardInfo.push([i, j, 0])
			}
		}
	}
	setCheckerboard();

	//改变棋盘格子颜色
	const changeCheckerboard = ([A, B], color) => {
		for (let i = 0; i < checkerboardInfo.length; i++) {
			let [x, y, num] = checkerboardInfo[i];
			if (A === x && B === y) {
				checkerboardInfo[i][2] = color;
				break
			}
		}
	};

	//清空棋盘颜色
	const clearCheckerboard = () => {
		for (let index = 0; index < checkerboardInfo.length; index++) {
			checkerboardInfo[index][2] = 0
		}
	};

	//构造方块
	const setSquare = () => {
		//失败后停止构造新的方块
		if (isFail.value) {
			return
		}
		isShowSquare.value = true
		moveSquareCoordinate = null
		randomSquareType();
		switch (squareTypeIndex) {

			//长方形
			case 0:
				moveSquareCoordinate = [
					[0, 6],
					[0, 7],
					[0, 8],
					[0, 9]
				]
				break;

				//三角形
			case 1:
				moveSquareCoordinate = [
					[0, 8],
					[0, 9],
					[0, 10],
					[1, 9]
				]
				break;

				//正方形
			case 2:
				moveSquareCoordinate = [
					[0, 7],
					[0, 8],
					[1, 7],
					[1, 8]
				]
				break;

				//z字型
			case 3:
				moveSquareCoordinate = [
					[0, 7],
					[0, 8],
					[1, 8],
					[1, 9]
				]
				break;

				//反z字型
			case 4:
				moveSquareCoordinate = [
					[0, 8],
					[0, 7],
					[1, 7],
					[1, 6]
				]
				break;

				//l型
			case 5:
				moveSquareCoordinate = [
					[0, 8],
					[0, 9],
					[0, 10],
					[1, 8]
				]
				break;

				//反l型
			case 6:
				moveSquareCoordinate = [
					[0, 8],
					[0, 9],
					[0, 10],
					[1, 10]
				]
				break;
		}
	}


	//判断是否碰到边界
	const judgeBoundary = (arr) => {
		if ((arr[0] < 0 || arr[0] > 24) || (arr[1] < 0 || arr[1] > 15)) {
			return true
		};
		return false
	}

	//查看方块是否碰到已经存在的方格中
	const judgeStabilitySquareCoordinate = (arr) => {
		//移动到最后一格时
		if (arr[0] === 24) {
			return true
		}
		for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
			if (stabilitySquareCoordinate[index][0] - 1 === arr[0] && stabilitySquareCoordinate[index][1] === arr[1]) {
				return true
			}
		};
		return false;
	}

	//得分
	const score = (arr) => {
		let num = 0;
		for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
			if (arr[0] === stabilitySquareCoordinate[index][0]) {
				num++;
				//等于16后满足当前销毁需求（直接跳出循环减少性能消耗）
				if (num === 16) {
					break;
				}
			}
		};
		if (num === 16) {
			//删除已经在该数组中凑齐能得分的行数
			let isScore = false
			for (let index = stabilitySquareCoordinate.length - 1; index > -1; index--) {
				if (arr[0] === stabilitySquareCoordinate[index][0]) {
					stabilitySquareCoordinate.splice(index, 1)
					isScore = true
				}
			}
			if (isScore) scoreNum.value = scoreNum.value + 100
			//将所有在销毁行上面的稳定方块移动至下一行去
			for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
				if (arr[0] > stabilitySquareCoordinate[index][0]) {
					stabilitySquareCoordinate[index][0]++
				}
			}
		}
	}

	//移动方块的指令
	const moveSquare = (num) => {
		if (!isShowSquare.value) {
			return
		};

		//移动
		for (let index = 0; index < moveSquareCoordinate.length; index++) {
			switch (num) {
				//键盘对应数字如下
				//40:下；37：左；39：右；
				case 37:
					moveSquareCoordinate[index][1] = moveSquareCoordinate[index][1] - 1
					break;
				case 39:
					moveSquareCoordinate[index][1] = moveSquareCoordinate[index][1] + 1
					break;
				case 40:
					moveSquareCoordinate[index][0] = moveSquareCoordinate[index][0] + 1
					break;
			};
		};

		//是否触碰到边界需要复原的标杆
		let flag1 = false;
		for (let index = 0; index < moveSquareCoordinate.length; index++) {
			if (judgeBoundary(moveSquareCoordinate[index])) {
				flag1 = true;
			}
		}

		//标杆满足后方块复位
		if (flag1) {
			for (let index = 0; index < moveSquareCoordinate.length; index++) {
				switch (num) {
					case 37:
						moveSquareCoordinate[index][1] = moveSquareCoordinate[index][1] + 1
						break;
					case 39:
						moveSquareCoordinate[index][1] = moveSquareCoordinate[index][1] - 1
						break;
					case 40:
						moveSquareCoordinate[index][0] = moveSquareCoordinate[index][0] - 1
						break;
				};
			};
		}

		//能否触碰到已稳定方块的标杆
		let flag2 = false;
		let coordinate = [];
		for (let index = 0; index < moveSquareCoordinate.length; index++) {
			if (judgeStabilitySquareCoordinate(moveSquareCoordinate[index])) {
				flag2 = true;
			}
		};

		//只要碰到了
		if (flag2) {

			//添加进入不移动的方块坐标数组
			for (let index = 0; index < moveSquareCoordinate.length; index++) {
				stabilitySquareCoordinate.push(moveSquareCoordinate[index]);
				if (moveSquareCoordinate[index][0] - 1 === 0) {
					//输了就跳出循环，不必再给已稳定的方块坐标添加新坐标了
					isFail.value = true;
					break
				}
			}

			//如果已经失败则停止移动
			if (isFail.value) {
				return
			}
			isShowSquare.value = false;

			//将移动方块中每个点坐标去做得分判断
			for (let index = 0; index < moveSquareCoordinate.length; index++) {
				score(moveSquareCoordinate[index]);
			}
		}

		clearCheckerboard();
		for (let index = 0; index < moveSquareCoordinate.length; index++) {
			changeCheckerboard(moveSquareCoordinate[index], 2)
		}
		for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
			changeCheckerboard(stabilitySquareCoordinate[index], 1)
		}
	};

	//检验工具（长方形）
	const detectionToolAboutRectangle = () => {
		if (moveSquareCoordinate[0][0] !== moveSquareCoordinate[1][0]) {
			//竖直的方块
			return 0
		}

		//横着的长方形方块
		return 1
	};

	//检验工具（三角形、l型、反l型）
	const detectionToolAboutTriangle = () => {

		//三角形有四种形态
		if ((moveSquareCoordinate[0][0] === moveSquareCoordinate[1][0] && moveSquareCoordinate[1][0] ===
				moveSquareCoordinate[2][0]) && moveSquareCoordinate[1][0] < moveSquareCoordinate[3][0]) {
			return 0
		}
		if ((moveSquareCoordinate[0][1] === moveSquareCoordinate[1][1] && moveSquareCoordinate[1][1] ===
				moveSquareCoordinate[2][1]) && moveSquareCoordinate[2][1] > moveSquareCoordinate[3][1]) {
			return 1
		}
		if ((moveSquareCoordinate[0][0] === moveSquareCoordinate[1][0] && moveSquareCoordinate[1][0] ===
				moveSquareCoordinate[2][0]) && moveSquareCoordinate[1][0] > moveSquareCoordinate[3][0]) {
			return 2
		}
		if ((moveSquareCoordinate[0][1] === moveSquareCoordinate[1][1] && moveSquareCoordinate[1][1] ===
				moveSquareCoordinate[2][1]) && moveSquareCoordinate[2][1] < moveSquareCoordinate[3][1]) {
			return 3
		}
	};

	//检验工具（z字形和反z字形）
	const detectionToolAboutZOr_Z = () => {
		if (moveSquareCoordinate[0][0] === moveSquareCoordinate[1][0]) {
			//z字形
			return 0
		}
		//n字形
		return 1
	};


	//切换方块形态(长方形)
	const toggleSquareShapeAboutRectangle = () => {

		//改变后的坐标点
		let arr = null;
		switch (detectionToolAboutRectangle()) {
			case 0:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 2]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 1:
				arr = [
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 2, moveSquareCoordinate[1][1]]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
		}
	};

	//切换方块形态(三角形)
	const toggleSquareShapeAboutTriangle = () => {
		//改变后的坐标点
		let arr = null;
		switch (detectionToolAboutTriangle()) {
			case 0:
				arr = [
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 1:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 2:
				arr = [
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 3:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
		}
	};

	//切换方块形态(z字型)
	const toggleSquareShapeAboutZ = () => {
		//改变后的坐标点
		let arr = null;
		switch (detectionToolAboutZOr_Z()) {
			case 0:
				arr = [
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 1:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] + 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
		}
	};

	//切换方块形态(反z字型)
	const toggleSquareShapeAbout_Z = () => {
		//改变后的坐标点
		let arr = null;
		switch (detectionToolAboutZOr_Z()) {
			case 0:
				arr = [
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 1:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
		}
	};

	//切换方块形态(l型)
	const toggleSquareShapeAboutL = () => {
		//改变后的坐标点
		let arr = null;
		switch (detectionToolAboutTriangle()) {
			case 0:
				arr = [
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 1:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1] + 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 2:
				arr = [
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] + 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 3:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
		}
	};

	//切换方块形态(反l型)
	const toggleSquareShapeAbout_L = () => {
		//改变后的坐标点
		let arr = null;
		switch (detectionToolAboutTriangle()) {
			case 0:
				arr = [
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 1:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1] - 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 2:
				arr = [
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0] - 1, moveSquareCoordinate[1][1] + 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
			case 3:
				arr = [
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] - 1],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1]],
					[moveSquareCoordinate[1][0], moveSquareCoordinate[1][1] + 1],
					[moveSquareCoordinate[1][0] + 1, moveSquareCoordinate[1][1] + 1]
				];
				for (let index = 0; index < arr.length; index++) {
					if (judgeBoundary(arr[index]) || judgeStabilitySquareCoordinate(arr[index])) {
						return
					}
				}
				moveSquareCoordinate = arr;
				clearCheckerboard();
				for (let index = 0; index < moveSquareCoordinate.length; index++) {
					changeCheckerboard(moveSquareCoordinate[index], 2)
				}
				for (let index = 0; index < stabilitySquareCoordinate.length; index++) {
					changeCheckerboard(stabilitySquareCoordinate[index], 1)
				}
				break;
		}
	};

	//切换方块形态
	const toggleSquareShape = (num) => {
		switch (num) {
			//长方形
			case 0:
				toggleSquareShapeAboutRectangle();
				break;

				//三角形
			case 1:
				toggleSquareShapeAboutTriangle();
				break;

				//z字型
			case 3:
				toggleSquareShapeAboutZ();
				break;

				//反z字型
			case 4:
				toggleSquareShapeAbout_Z();
				break;

				//l型
			case 5:
				toggleSquareShapeAboutL();
				break;

				//反l型
			case 6:
				toggleSquareShapeAbout_L();
				break;
		}
	}

	//事件（方便后期摘除事件）
	const listener = (event) => {
		//只监听上下左右四个按键
		const keyCodeArr = [37, 39, 40];
		if (keyCodeArr.includes(event.keyCode)) {
			//监听方向键变化（执行方块移动方向）
			moveSquare(event.keyCode);
		}

		//上键即切换状态
		if (event.keyCode === 38) {
			//监听到按下k值时切换方块形态
			toggleSquareShape(squareTypeIndex);
		}
	}

	//在window上挂载事件监听器
	window.addEventListener("keydown", listener);

	//定时器(一直会有一个下移方块指令执行)
	let timer = setInterval(() => {
		moveSquare(40)
	}, 500);

	watchEffect(() => {
		if (isFail.value) {
			clearInterval(timer);
			window.removeEventListener('keydown', listener)
		}
	})
	//监听到方块消失后就执行
	watchEffect(() => {
		if (isShowSquare.value === false) {
			setSquare();

			//在重置方块后会渲染一次颜色（这样第一行才能看见）
			for (let index = 0; index < moveSquareCoordinate.length; index++) {
				changeCheckerboard(moveSquareCoordinate[index], 2)
			}
		}
	})
</script>

<style lang="scss" scoped>
	.col,.col1 {
		display: flex;
		justify-content: center;

		image {
			width: 120rpx;
			height: 120rpx;
			transition: all .2s; 
		}
		image:active{
			transform: scale(0.95);
		}
	}
	
	.col1{
		image:last-child{
			margin-left: 120rpx;
		}
	}
</style>
