<template>
	<view class="dice-container" :style="[containerStyle]">
		<!-- 骰子容器 -->
		<view class="dice" :style="[diceStyle]">
			<!-- 六个面 -->
			<view v-for="(face, index) in processedFaces" :key="index" class="dice-face" :class="[getFaceClass(index)]"
				:style="[getFaceStyle(index)]">
				{{ face }}
			</view>
		</view>
	</view>
</template>

<script>
	export default {
		props: {
			// 骰子六个面的文案
			faces: {
				type: Array,
				default: () => ['1', '2', '3', '4', '5', '6'],
				// validator: arr => arr.length === 6
			},
			// 骰子大小（单位：rpx）
			size: {
				type: Number,
				default: 300
			},
			backgroundColor: {
				type: String,
				default: '#4CAF50'
			},
			fontSize: {
				type: Number,
				default: 40
			},
		},
		data() {
			return {
				rotateX: 10,
				rotateY: 20,
				rotateZ: 30,
				isRolling: false,
				result: null,
				lastResult: null,
				history: [],
				duration: 2000,
				lastTurns: null, // 用于记录上一次滚动的圈数
			};
		},
		computed: {
			// 处理后的faces数组（自动截断/填充）
			processedFaces() {
				const result = this.faces.slice(0, 6); // 截断前6个
				// 补充空字符串至6个元素
				while (result.length < 6) {
					result.push('空');
				}
				return result;
			},
			// 新增：容器样式
			containerStyle() {
				// 容器宽高为骰子尺寸的1.5倍，确保旋转时不会超出容器
				const containerSize = this.size * 1.5;
				return {
					width: `${containerSize}rpx`,
					height: `${containerSize}rpx`,
				};
			},
			// 动态骰子样式（宽高、旋转、过渡）
			diceStyle() {
				return {
					width: this.size + 'rpx',
					height: this.size + 'rpx',
					transform: `rotateX(${this.rotateX}deg) rotateY(${this.rotateY}deg) rotateZ(${this.rotateZ}deg)`,
					transition: this.isRolling ? `transform ${this.duration}ms ease-out` : 'none'
				};
			},
		},
		mounted() {

		},
		methods: {
			// 获取骰子面类名
			getFaceClass(index) {
				const classes = ['front', 'back', 'right', 'left', 'top', 'bottom'];
				return classes[index];
			},
			// 获取骰子面样式（位置、大小）
			getFaceStyle(index) {
				const sizeHalf = this.size / 2;
				const transforms = [
					`translateZ(${sizeHalf}rpx)`, // front
					`rotateX(180deg) translateZ(${sizeHalf}rpx)`, // back
					`rotateY(90deg) translateZ(${sizeHalf}rpx)`, // right
					`rotateY(-90deg) translateZ(${sizeHalf}rpx)`, // left
					`rotateX(90deg) translateZ(${sizeHalf}rpx)`, // top
					`rotateX(-90deg) translateZ(${sizeHalf}rpx)` // bottom
				];

				// 获取当前面的内容
				let dynamicSize=this.fontSize;
				
				const content = this.processedFaces[index] || '';
				const len = content.length;
				if(len>2){
					// 设置字体大小的动态计算逻辑
					const baseSize = this.fontSize;
					const minSize = 20; // 最小字体大小
					const charReduction = 5; // 每个字符减少的字体大小

					dynamicSize = baseSize - (len * charReduction);
					dynamicSize = Math.max(minSize, dynamicSize); // 防止字体过小
				}
				// console.log(this.fontSize,dynamicSize,content,content.length)
				return {
					width: this.size + 'rpx',
					height: this.size + 'rpx',
					transform: transforms[index],
					backgroundColor: this.backgroundColor,
					fontSize: dynamicSize + 'rpx'
				};
			},
			// 获取随机结果（0~5）
			getRandomResult() {
				return Math.floor(Math.random() * 6);
			},
			// 开始滚动骰子（可被外部调用）
			startRolling() {
				if (this.isRolling) return;

				this.isRolling = true;
				this.result = null;
				// 1. 随机选择结果（0-5对应六个面）
				let targetFace = this.getRandomResult();
				// 2. 为每个面定义精确的摄像机朝向角度
				const faceAngles = [{
						x: 0,
						y: 10,
						z: 10
					}, // 1点面（正面）
					{
						x: 180,
						y: 10,
						z: 10
					}, // 6点面（后面）
					{
						x: 10,
						y: -90,
						z: 10
					}, // 5点面（右面）
					{
						x: 0,
						y: 90,
						z: 10
					}, // 2点面（左面）
					{
						x: -90,
						y: 10,
						z: 10
					}, // 4点面（下面）
					{
						x: 90,
						y: 10,
						z: 10
					} // 3点面（上面）
				];
				// 3. 计算最终旋转角度（包含多圈旋转）
				let turns = 3 + Math.floor(Math.random() * 3); // 3-5圈旋转
				// 判断是否与上一次相同
				let wasSameTurn = turns === this.lastTurns;
				if (wasSameTurn) {
					turns += 3 + Math.floor(Math.random() * 3); // 额外圈数
				}

				// 4. 应用新旋转角度
				this.rotateX = turns * 360 + faceAngles[targetFace].x;
				this.rotateY = turns * 360 + faceAngles[targetFace].y;
				this.rotateZ = turns * 360 + faceAngles[targetFace].z;

				// 5. 添加小幅度随机偏移（±5°）使动画更自然
				this.rotateX += (Math.random() * 10) - 5;
				this.rotateY += (Math.random() * 10) - 5;
				this.rotateZ += (Math.random() * 10) - 5;

				this.lastTurns = turns;

				// 6. 动画结束后确定结果
				setTimeout(() => {
					this.isRolling = false;
					// 添加细微抖动效果（可选）
					this.rotateX += (Math.random() * 2) - 1;
					this.rotateY += (Math.random() * 2) - 1;
					this.rotateZ += (Math.random() * 2) - 1;

					this.result = this.processedFaces[targetFace];
					// 回调
					this.$emit('result', this.result);
				}, this.duration);
			}
		}
	};
</script>

<style>
	/* 容器样式 */
	.dice-container {
		perspective: 1000rpx;
		display: flex;
		flex-direction: column;
		align-items: center;
		padding: 40rpx;
		overflow: visible;
	}

	/* 骰子整体样式 */
	.dice {
		position: relative;
		transform-style: preserve-3d;
		transform: rotateX(35deg) rotateY(45deg);
		/* 初始视角 */
		transition: transform 1s ease-out;
	}

	/* 面样式 */
	.dice-face {
		position: absolute;
		/* background: #4CAF50; */
		border: 4rpx solid #fff;
		border-radius: 10rpx;
		color: #fff;
		/* font-size: 40rpx; */
		font-weight: bold;
		display: flex;
		align-items: center;
		justify-content: center;
		text-align: center;
		box-shadow:
			inset 0 0 20rpx rgba(255, 255, 255, 0.3),
			0 0 20rpx rgba(0, 0, 0, 0.2);
	}
</style>