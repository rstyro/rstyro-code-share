<template>
	<view class="container">
		<!-- 页面头部 -->
		<view class="header">
			<view class="logo">
				<cl-icon type="icon-qinglv" size="30" color="#fff" />
				<text class="logo-text">情侣飞行棋</text>
				<text class="version">{{taskData.version}}</text>
			</view>
			<view class="tips">
				<view class="item" @click="operateHelp"><cl-icon type="icon-wenhao" size="20" color="#fff" /></view>
				<view v-if="!isDevOrTrial" class="item" @click="operateSetting"><cl-icon type="icon-setting" size="23"
						color="#fff" /></view>
			</view>
		</view>

		<!-- 游戏主体 -->
		<view class="game-content">
			<!-- 棋盘容器 -->
			<view class="chessboard-container">
				<view class="chessboard">
					<!-- 动态生成棋盘格子 -->
					<block v-for="(row, rowIndex) in checkerboard" :key="rowIndex">
						<block v-for="(cell, colIndex) in row" :key="cell.id">
							<view class="cell" :class="{ show: cell.pathNum > 0,
											start:cell.type==='start',
											end:cell.type==='end',
											task:cell.type==='task',
											warn:cell.type==='warn',
											trap:cell.type==='trap',
											normal:cell.type==='normal' }" :style="{ backgroundColor: cell.bgColor }">
								<view v-if="cell.pathNum>0">

									<view class="image-container glowPulse" v-if="cell.type==='trap'">
										<cl-icon type="icon-bomb" size="30" color="#111"></cl-icon>
									</view>

									<view class="image-container scaleAnimation" v-if="cell.type==='task'">
										<cl-icon type="icon-qinglv" size="20" color="#ff758c"></cl-icon>
									</view>

									<view class="image-container pulse" v-if="cell.type==='warn'">
										<cl-icon type="icon-baojingshu" size="20" color="#f44336"></cl-icon>
									</view>

									<view class="image-container" v-if="cell.type==='start'">
										<cl-icon type="icon-qidian" size="50" color="#4CAF50"></cl-icon>
									</view>
									<view class="image-container" v-else-if="cell.type==='end'">
										<cl-icon type="icon-zhongdian" size="40" color="#FF5722"></cl-icon>
									</view>
									<view v-else class="step"> {{ cell.pathNum }}</view>
								</view>
							</view>
						</block>
					</block>
				</view>

				<!-- 人物 -->
				<view class="character man" :style="[manStyle]">
					<image class="image" src="/static/images/fly/man.png" mode="aspectFill" />
				</view>

				<view class="character woman" :style="[womanStyle]">
					<image class="image" src="/static/images/fly/woman.png" mode="aspectFill" />
				</view>
			</view>

			<!-- 控制面板 -->
			<view class="control-panel" :class="{man:isCurrentPlayerMan,woman:!isCurrentPlayerMan}">
				<view class="dice-container" :class="{ rolling: isRolling }" @click="rollDice">
					<cl-dice ref="flyDice" @result="getDiceData" :size="100" backgroundColor="#ff758c"></cl-dice>
				</view>

				<view class="operateBtn">
					<view class="reload-btn" @click="reloadBoard">
						<cl-icon type="icon-shuaxin" size="26" color="#fff" class="icon" />
						<text>刷新棋盘</text>
					</view>

					<view class="player-turn">
						<text>{{ isCurrentPlayerMan? '男生' : '女生' }}回合</text>
					</view>

				</view>
			</view>

		</view>

		<!-- 任务弹窗 -->
		<cl-fly-task :show="showTask" :isMan="manTask" :taskContent="currentTask" @close="handleTaskResult(false)"
			@complete="handleTaskResult(true)" />

		<!-- 玩法介绍 -->
		<cl-fly-help :show="showHelp" @close="operateHelp" />
		<!-- 设置 -->
		<cl-fly-settings :show="showSetting" :options="taskOptions" type="fly" @close="operateSetting"
			@apply="changeVersion" @edit="goEdit" />

		<!-- 编辑组件 -->
		<cl-edit-task :show="showEdit" :data="taskData" @save="saveEdit" @close="closeEdit"
			title="飞行棋任务编辑"
			versionLabel="版本名称"
			listTitle="任务列表"
			itemLabel="任务"
			itemPlaceholder="请输入任务内容"
			importTitle="导入任务"
			importButtonText="导入任务"
			importDescription="请输入任务列表，每个任务占一行"
			importPlaceholder="输入任务1\n输入任务2\n输入任务3" />

		<!-- 游戏结束弹窗 -->
		<view v-if="gameOver" class="game-over" :class="{show:gameOver}">
			<text class="winner-text">{{ winnerText }}</text>
			<button class="restart-btn" @click="restartGame">重新开始</button>
		</view>

		<view class="setting">

		</view>
	</view>
</template>

<script>
	// 导入数据文件
	import {
		taskMap
	} from '@/data/flyTask.js';

	export default {
		data() {
			return {
				isDevOrTrial: true,
				showEdit: false,
				curKey: 'normal',
				flyTaskMap: {},
				// 任务数据
				taskData: {
					version: '甜蜜版',
					tasks: ['问对方一个真心话']
				},
				// 节点个数
				nodeNum: 30,
				// 
				taskNum: 12,
				// 对方做任务
				warnNum: 6,
				// 炸弹，回到原点
				trapNum: 3,
				// 棋盘数据
				checkerboard: [],
				// 三种随机任务颜色
				taskColors: ['#ffe0b2', '#ffccbc', '#c8e6c9'],
				normalColors: ['#FFE6F2', '#FFF0E6', '#E6F7FF'],
				// 保存起点单元格
				srcCell: {
					position: [0, 0]
				},
				// 玩家位置
				manData: {
					name:'骑士先生',
					top: 0,
					left: 0,
					scale: 1,
					rotation: 0,
					prePath: [],
					nextPath: [],
					time: 0.5
				},
				womanData: {
					name:'公主殿下',
					top: 0,
					left: 0,
					scale: 1,
					rotation: 0,
					prePath: [],
					nextPath: [],
					time: 0.5
				},
				// 当前是否是男玩家回合
				isCurrentPlayerMan: false,
				// 是否是男方做任务
				manTask: false,
				isRolling: false,
				gameOver: false,
				showHelp: false,
				// 弹窗状态
				showTask: false,
				showSetting: false,
				currentTask: '轻轻捏对方的脸颊',
				winnerText: '💞 爱情棋盘上，你们都是赢家~'
			};
		},

		computed: {
			manStyle() {
				return {
					top: `calc(100%/7*${this.manData.top+0.2})`,
					left: `calc(100%/7*${this.manData.left+0.1})`,
					transform: `translate(0, 0%) rotate(${this.manData.rotation}deg) scale(${this.manData.scale})`,
					transition: `all ${this.manData.time}s ease-in-out`
				}
			},
			womanStyle() {
				return {
					top: `calc(100%/7*${this.womanData.top+0.2})`,
					left: `calc(100%/7*${this.womanData.left+0.2})`,
					transform: `translate(0, 0%) rotate(${this.womanData.rotation}deg) scale(${this.womanData.scale})`,
					transition: `all ${this.womanData.time}s ease-in-out`
				}
			},
			// 编辑任务的选项
			taskOptions() {
				// 示例：遍历所有键，组装选项
				let options = [];
				Object.keys(this.flyTaskMap).forEach(key => {
					let item = {
						label: this.flyTaskMap[key].version,
						value: key
					};
					if (key === 'cp') {
						item.hot = true;
						item.hotText = 'HOT';
					} else if (key === 'love') {
						item.hot = true;
						item.hotText = '999+';
					} else if (key === 'shame') {
						item.hot = true;
						item.hotText = '火爆';
					} else if (key === 'shameUp') {
						item.hot = true;
						item.hotText = '18禁';
					}
					options.push(item)
				});
				return options;
			}
		},
		onShow() {
			this.curKey = 'normal';
		},
		onReady() {
			this.initGame();
			this.onloadData();
			uni.$emit('global-user-interaction');
		},
		onLoad() {
		},
		methods: {
			getRandomTask() {
				if (this.taskData.tasks.length === 0) return null;
				const num = Math.floor(Math.random() * this.taskData.tasks.length);
				return this.taskData.tasks[num];
			},
			// 加载任务数据
			onloadData() {
				var stopMap = this.store_fly_task_map;
				if (stopMap && Object.keys(stopMap).length > 0) {
					this.flyTaskMap = JSON.parse(stopMap);
				} else {
					this.flyTaskMap = taskMap;
				}
				uni.$u.vuex('store_fly_task_map', JSON.stringify(this.flyTaskMap));
				this.taskData = this.flyTaskMap[this.curKey];
			},
			closeEdit() {
				this.showEdit = false;
			},
			// 保存编辑后的数据
			saveEdit(resData) {
				console.log("resData=", resData);
				this.flyTaskMap[this.curKey] = resData;
				this.taskData = resData;
				// 保存缓存
				uni.$u.vuex('store_fly_task_map', JSON.stringify(this.flyTaskMap));
			},
			// 摇骰子
			rollDice() {
				if (this.isRolling) {
					return false;
				}
				if (this.$refs.flyDice) {
					uni.$emit('global-user-interaction');
					this.$soundManager.play("touzi");
					this.$refs.flyDice.startRolling();
					this.currentTask = this.getRandomTask();
				} else {
					console.error('Dice 组件未挂载');
				}
			},
			goEdit(newVersion) {
				this.curKey = newVersion;
				this.taskData = this.flyTaskMap[newVersion];
				this.showEdit = true;
			},
			// 切换版本
			changeVersion(newVersion) {
				this.taskData = taskMap[newVersion];
			},
			operateSetting() {
				this.showSetting = !this.showSetting;
			},
			operateHelp() {
				this.showHelp = !this.showHelp;
			},
			reloadBoard() {
				this.initGame();
			},
			// 重新开始
			restartGame() {
				this.initGame();
			},
			// 封装随机数生成器
			getRandomStep(isCompleted) {
				return isCompleted ?
					Math.floor(Math.random() * 3) + 1 // 完成：1-3步
					:
					Math.floor(Math.random() * 3) + 3; // 未完成：3-5步
			},
			// 统一处理任务结果
			handleTaskResult(isCompleted) {
				const playerData = this.manTask ? this.manData : this.womanData;
				const steps = this.getRandomStep(isCompleted);
				this.move(playerData, steps, isCompleted, false);
				this.closeTask();
			},
			closeTask() {
				this.showTask = false;
				this.isCurrentPlayerMan = !this.isCurrentPlayerMan;
			},
			// 摇骰子得到结果，进行动画移动
			getDiceData(diceNum) {
				// diceNum=6;
				let playerData = this.isCurrentPlayerMan ? this.manData : this.womanData;
				this.move(playerData, diceNum);
			},
			// 移动人物
			async move(playerData, moveNum, shouldMoveNext = true, valid = true) {
				this.isRolling = true;
				try {
					for (var i = 0; i < moveNum; i++) {
						if (this.gameOver) break;
						if (shouldMoveNext) {
							const isGameEnd = this.moveNext(playerData);
							if (isGameEnd) {
								if (i === moveNum - 1) {
									this.winnerText =
										`💋 亲亲一下~ ${playerData.name}赢啦！快来执行秘密惩罚！`;
									this.gameOver = true;
									uni.$emit('global-user-interaction');
									this.$soundManager.play("sound4");
								} else {
									shouldMoveNext = false;
								}
							}
						} else {
							this.moveBack(playerData);
						}
						if (!shouldMoveNext && this.isSrcPosition(playerData)) {
							break;
						}
						// 播放音乐
						this.playMusic();
						// 每次移动后等待 700ms
						await this.sleep(700);
					}
					// 是否是正常的移动（惩罚和奖励不触发该格子效果）
					if (valid) {
						const targetCell = this.checkerboard[playerData.top]?.[playerData.left];
						if (targetCell.type === 'task' || targetCell.type === 'warn') {
							this.showTask = true;
							this.manTask = (this.isCurrentPlayerMan && targetCell.type === 'task') ||
								(!this.isCurrentPlayerMan && targetCell.type === 'warn')
						} else if (targetCell.type === 'trap') {
							// 炸弹，直接回到原点
							this.playMusic('boom');
							this.resetUserData(this.srcCell, playerData, 3, 360, 1.5);
							this.isCurrentPlayerMan = !this.isCurrentPlayerMan;
						} else {
							this.isCurrentPlayerMan = !this.isCurrentPlayerMan;
						}
					}
				} catch (e) {
					console.log("移动报错，err=", e);
				} finally {
					this.isRolling = false;
				}
			},
			playMusic(soundKey = 'sound5') {
				// 播放音效
				const sound = this.$soundManager.getSound(soundKey);
				if (!sound) return;
				// 移除之前的监听避免重复
				sound.offEnded();
				sound.currentTime = 0;
				sound.play();
			},
			// 向前
			moveNext(userData) {
				const [row, col] = userData.nextPath;
				const targetCell = this.checkerboard[row]?.[col];
				if (!targetCell) {
					console.warn("目标格子不存在或越界", {
						row,
						col
					});
					uni.$u.toast('目标格子不存在或越界');
					return;
				}
				return this.resetUserData(targetCell, userData);
			},
			// 回退
			moveBack(userData) {
				if (this.isSrcPosition(userData)) {
					// 如果已经是原点了就没法回退了
					return;
				}
				const [row, col] = userData.prePath;
				const targetCell = this.checkerboard[row]?.[col];
				if (!targetCell) {
					console.warn("目标格子不存在或越界", {
						row,
						col
					});
					uni.$u.toast('目标格子不存在或越界');
					return;
				}
				return this.resetUserData(targetCell, userData);
			},
			isSrcPosition(userData){
				if (this.srcCell.position[0] === userData.top &&
					this.srcCell.position[1] === userData.left) {
					return true;
				}
				return false;
			},
			// 工具函数：等待指定毫秒
			sleep(ms) {
				return new Promise(resolve => setTimeout(resolve, ms));
			},
			// 初始化游戏
			initGame() {
				this.generatePathArray();
				// 初始化角色位置
				this.resetUserData(this.srcCell, this.manData);
				this.resetUserData(this.srcCell, this.womanData);
				this.isRolling = false;
				this.gameOver = false;
				this.showTask = false;
			},
			// 更新棋子数据
			resetUserData(cellData, userData, scale = 1.3, rotation = 0, time = 0.5) {
				// console.log("cellData=", cellData, cellData.type);
				const [top, left] = cellData.position || [0, 0];
				const nextPath = cellData.nextPath || [];
				const prePath = cellData.prePath || [];
				// 设置缩放
				// const scale = 1.3;
				Object.assign(userData, {
					top,
					left,
					scale,
					rotation,
					time,
					nextPath,
					prePath
				});
				let timeout = (userData.time * 1000)

				setTimeout(() => {
					// 恢复缩放
					userData.scale = 1
					userData.time = 0.5
					userData.rotation = 0
				}, timeout);
				return cellData.type === 'end';
			},
			// 初始化路径
			generatePathArray() {
				const size = 7;
				const requiredLength = this.nodeNum;
				// 初始化棋盘并分配 id
				let newCheckerboard = this.initCheckerboard(size);
				// 生成路径
				const path = this.generateLongContinuousPath(size, requiredLength);
				if (!path || path.length < requiredLength) {
					console.error('无法生成足够长的连续路径');
					return;
				}

				// 收集所有可修改的普通节点坐标（排除起点和终点）
				const modifiableNodes = [];
				path.forEach((coord, index) => {
					// 只收集中间节点（非起点和终点）
					if (index > 0 && index < path.length - 1) {
						modifiableNodes.push(coord);
					}
				});
				// 随机选择节点修改类型
				this.randomizeNodeTypes(modifiableNodes, newCheckerboard);

				// 填充路径节点
				path.forEach((coord, index) => {
					let [row, col] = coord;
					// 使用已修改的类型

					let type = newCheckerboard[row][col].type || 'normal';
					if (type === 'empty') {
						type = 'normal';
					}
					let text = `路径${index + 1}`;

					if (index === 0) {
						type = 'start';
						text = '起点';
					} else if (index === path.length - 1) {
						type = 'end';
						text = '终点';
					}

					newCheckerboard[row][col] = {
						...newCheckerboard[row][col], // 保留 id
						type,
						text,
						position: [row, col],
						pathNum: index + 1,
						prePath: index > 0 ? [...path[index - 1]] : [],
						nextPath: index < path.length - 1 ? [...path[index + 1]] : [],
						bgColor: this.getNodeColor(type)
					};
					if (index === 0) {
						this.srcCell = {
							...newCheckerboard[row][col]
						};
					}
				});
				// 更新棋盘数据
				this.$set(this, 'checkerboard', newCheckerboard);
			},
			// 随机修改节点类型
			randomizeNodeTypes(modifiableNodes, checkerboard) {
				// 复制可修改节点数组避免污染原数据
				const nodes = [...modifiableNodes];
				const totalModifiable = nodes.length;

				// 计算实际可修改数量（不超过节点总数）
				const taskCount = Math.min(this.taskNum, totalModifiable);
				const warnCount = Math.min(this.warnNum, totalModifiable - taskCount);
				const trapCount = Math.min(this.trapNum, totalModifiable - taskCount - warnCount);

				// 随机打乱节点顺序
				this.shuffleArray(nodes);

				// 设置任务节点
				for (let i = 0; i < taskCount; i++) {
					const [row, col] = nodes[i];
					checkerboard[row][col].type = 'task';
				}
				// 设置陷阱节点
				for (let i = taskCount; i < taskCount + warnCount; i++) {
					const [row, col] = nodes[i];
					checkerboard[row][col].type = 'warn';
				}
				// 设置陷阱节点
				for (let i = taskCount + warnCount; i < taskCount + warnCount + trapCount; i++) {
					const [row, col] = nodes[i];
					checkerboard[row][col].type = 'trap';
				}
			},
			// 数组随机排序（Fisher-Yates洗牌算法）
			shuffleArray(array) {
				for (let i = array.length - 1; i > 0; i--) {
					const j = Math.floor(Math.random() * (i + 1));
					[array[i], array[j]] = [array[j], array[i]];
				}
				return array;
			},
			// 新增：根据节点类型返回颜色
			getNodeColor(type) {
				// 使用组件中定义的配色数组
				const {
					taskColors,
					normalColors
				} = this;

				switch (type) {
					case 'start':
						return '#4CAF50'; // 绿色
					case 'end':
						return '#FF5722'; // 橙色
					case 'trap':
						return '#E91E63'; // 粉色
					case 'task':
						// 从任务颜色数组中随机选择
						// return taskColors[Math.floor(Math.random() * taskColors.length)];
						return '#ffccbc';
					case 'warn':
						return '#ffccbc';
					case 'normal':
					default:
						// 从普通颜色数组中随机选择
						// return normalColors[Math.floor(Math.random() * normalColors.length)];
						return '#FFF0E6';
				}
			},
			// 初始化棋盘并分配 id
			initCheckerboard(size) {
				let board = Array(size).fill().map((_, row) =>
					Array(size).fill().map((_, col) => {
						const id = row * size + col + 1; // id 从 1 到 49
						return {
							id,
							type: 'empty',
							text: '',
							pathNum: 0,
							prePath: [],
							nextPath: []
						};
					})
				);
				return board;
			},

			// 生成长连续路径（DFS + 回溯 + 分支优先）
			generateLongContinuousPath(size, minLength) {
				const directions = [
					[-1, 0],
					[1, 0],
					[0, -1],
					[0, 1]
				]; // 上下左右
				const visited = Array(size).fill().map(() => Array(size).fill(false));
				const path = [];

				// 随机起点
				let row = Math.floor(Math.random() * size);
				let col = Math.floor(Math.random() * size);
				path.push([row, col]);
				visited[row][col] = true;
				// 保存起点坐标
				this.startPosition = [row, col];
				// 使用栈结构辅助回溯
				const stack = [
					[row, col]
				];
				const branchPoints = [];

				const maxAttempts = 1000;
				let attempts = 0;

				while (path.length < minLength && attempts++ < maxAttempts) {
					const current = stack[stack.length - 1];
					const possibleMoves = [];

					// 找出所有未访问的相邻格子，并记录其周围未访问格子数量
					directions.forEach(dir => {
						const newRow = current[0] + dir[0];
						const newCol = current[1] + dir[1];
						if (
							newRow >= 0 && newRow < size &&
							newCol >= 0 && newCol < size &&
							!visited[newRow][newCol]
						) {
							possibleMoves.push({
								coord: [newRow, newCol],
								branches: directions.filter(d => {
									const r = newRow + d[0];
									const c = newCol + d[1];
									return r >= 0 && r < size && c >= 0 && c < size && !visited[r][
										c
									];
								}).length
							});
						}
					});

					// 按照分支数排序（贪心策略：先走分支多的）
					possibleMoves.sort((a, b) => b.branches - a.branches);

					if (possibleMoves.length > 0) {
						// 选择分支最多的路径
						const next = possibleMoves[0].coord;
						path.push(next);
						visited[next[0]][next[1]] = true;
						stack.push(next);
						branchPoints.push([...stack]); // 记录分支点
					} else {
						// 无路可走，尝试回退
						stack.pop();
						if (stack.length === 0) {
							// 所有分支都走完，无法生成足够长的路径
							if (branchPoints.length === 0) {
								return null;
							}
							// 从最近的分支点重新开始
							const newStart = branchPoints.pop();
							if (newStart.length === 0) return null;
							const lastPoint = newStart[newStart.length - 1];
							stack.length = 0;
							stack.push(...newStart);
							path.length = newStart.length;
						} else {
							path.pop();
						}
					}
				}

				return path;
			}
		}
	};
</script>

<style lang="scss" scoped>
	.container {
		background: linear-gradient(135deg, #ffd6e7, #c2e9fb);
		padding: 20rpx;
		min-height: 100vh;
		box-shadow: 0 15px 35px rgba(0, 0, 0, 0.2);
		overflow: hidden;
		position: relative;
	}

	.header {
		background: linear-gradient(to right, #ff7eb3, #ff758c);
		padding: 20rpx 25rpx;
		display: flex;
		justify-content: space-between;
		align-items: center;
		color: white;
		position: relative;
		z-index: 1;
		border-radius: 16px 16px 0 0;
		box-shadow: 0 4px 12px rgba(255, 117, 140, 0.4);

		.logo {
			display: flex;
			align-items: center;
			gap: 12rpx;
			text-shadow: 0 1px 3px rgba(0, 0, 0, 0.2);


			.logo-text {
				color: white;
				font-size: 48rpx;
				font-weight: bold;
				letter-spacing: 1px;
				text-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
			}

			.version {
				background: white;
				color: #ff758c;
				padding: 10rpx 20rpx;
				border-radius: 20px;
				font-size: 25rpx;
				font-weight: bold;
				margin-left: 10px;
				box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
			}

		}

		.tips {
			display: flex;
			gap: 10rpx;

			.item {
				background: rgba(255, 255, 255, 0.25);
				width: 60rpx;
				height: 60rpx;
				border-radius: 50%;
				display: flex;
				align-items: center;
				justify-content: center;
				font-weight: bold;
				font-size: 18px;
				transition: all 0.3s;
				cursor: pointer;
				box-shadow: 0 2px 5px rgba(0, 0, 0, 0.15);

				&:hover {
					background: rgba(255, 255, 255, 0.35);
					transform: translateY(-2px);
				}

				&:nth-child(2) {
					font-size: 22px;
				}
			}
		}

	}

	.game-content {
		position: relative;
	}

	.chessboard-container {
		background: #f8f4ff;
		border-radius: 0 0 16px 16px;
		padding: 10rpx;
		position: relative;
		box-shadow: inset 0 0 15px rgba(0, 0, 0, 0.1);
		margin-bottom: 20px;

		.image-container {
			width: 60rpx;
			height: 60rpx;
			position: relative;
			display: flex;
			align-items: center;
			justify-content: center;
			margin: 0 auto;



		}



		.image {
			width: 100%;
			height: 100%;
			border-radius: 50%;
		}
	}

	.chessboard {
		display: grid;
		grid-template-columns: repeat(7, 1fr);
		gap: 10rpx;
		margin: 0 auto;
		position: relative;
	}

	.cell {
		height: 100rpx;
		border-radius: 10px;
		display: flex;
		justify-content: center;
		align-items: center;
		position: relative;
		font-weight: bold;
		font-size: 18px;
		color: #555;
		transition: all 0.3s;

		.step {
			position: absolute;
			bottom: 5px;
			right: 5px;
			font-size: 14px;
			opacity: 0.8;
		}

	}

	.cell.show {
		box-shadow: 0 3px 6px rgba(0, 0, 0, 0.1);
	}

	.cell.active {
		transform: scale(0.95);
		box-shadow: 0 0 15px rgba(255, 117, 140, 0.6);
	}

	.cell.start {
		background: linear-gradient(135deg, #a8edea, #fed6e3);
	}

	.cell.end {
		background: linear-gradient(135deg, #ffafbd, #ffc3a0);
	}

	.cell.normal {
		// background: #e0f7fa;
	}

	.cell.task,
	.cell.warn,
	.cell.trap {
		color: #fff;
	}


	.cell-text {
		position: absolute;
		bottom: 5px;
		right: 5px;
		font-size: 14px;
		opacity: 0.8;
	}

	.character {
		position: absolute;
		width: 70rpx;
		height: 70rpx;
		border-radius: 50%;
		display: flex;
		justify-content: center;
		align-items: center;
		transition: all 0.5s ease-in-out;
		z-index: 5;
		box-shadow: 0 3px 10px rgba(0, 0, 0, 0.2);
	}

	.character.man {
		background: linear-gradient(135deg, #6a93cb, #a4bfef);
		color: white;
		transform: translate(0, 0);
	}

	.character.woman {
		background: linear-gradient(135deg, #ff9a9e, #fad0c4);
		color: white;
		transform: translate(0, 0);
	}

	.control-panel {
		border-radius: 15px;
		padding: 10rpx 20px;
		display: flex;
		flex-direction: column;
		align-items: center;
		box-shadow: 0 5px 15px rgba(0, 0, 0, 0.1);



		.operateBtn {
			display: flex;
			width: 100%;
			justify-content: space-between;
			gap: 15px;

			.reload-btn {
				// background: linear-gradient(to right, #ffafbd, #ffc3a0);
				background-color: #EE809E;
				color: white;
				padding: 8px 20px;
				border-radius: 15px;
				font-weight: bold;
				text-align: center;
				font-size: 18px;
				cursor: pointer;
				box-shadow: 0 5px 15px rgba(255, 175, 189, 0.3);
				transition: all 0.3s;

				.icon {
					margin-right: 10rpx;
				}
			}

			.player-turn {
				color: white;
				padding: 8px 20px;
				border-radius: 30px;
				font-weight: bold;
				margin-top: 10px;
				display: flex;
				align-items: center;
				gap: 8px;
			}

		}
	}

	.man {
		background: linear-gradient(135deg, #6a93cb, #a4bfef);
	}

	.woman {
		background: linear-gradient(to right, #ff9a9e, #fad0c4);
	}

	.game-over {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		background: radial-gradient(circle, rgba(0, 0, 0, 0.9) 0%, rgba(0, 0, 0, 0.95) 100%);
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		z-index: 200;
		opacity: 0;
		pointer-events: none;
		transition: opacity 0.6s;
	}

	.game-over.show {
		opacity: 1;
		pointer-events: all;
	}

	.winner-text {
		/* 动态属性 */
		--text-color: white;
		/* 使用CSS变量便于统一调整 */
		--shadow-size: 20px;

		/* 排版 */
		font-size: clamp(2.5rem, 8vw, 4rem);
		/* 响应式字体尺寸 */
		margin: 0 auto 2.5rem;
		text-align: center;
		font-weight: 900;
		/* 更厚重的字体权重 */
		letter-spacing: 1px;
		/* 增加字母间距提升可读性 */

		/* 视觉效果 */
		color: var(--text-color);
		text-shadow: 0 0 var(--shadow-size) rgba(255, 255, 255, 0.85);
		position: relative;
		padding: 1.25rem 2.5rem;
		background: linear-gradient(135deg,
				/* 调整渐变角度 */
				rgba(255, 255, 255, 0.15) 0%,
				rgba(255, 255, 255, 0.08) 100%);
		border-radius: 25px;
		border: 1px solid rgba(255, 255, 255, 0.25);
		/* 更强的边框对比 */
		backdrop-filter: blur(4px);
		/* 半透明模糊效果 */

		/* 3D立体阴影 */
		box-shadow:
			0 12px 35px rgba(0, 0, 0, 0.4),
			inset 0 4px 15px rgba(255, 255, 255, 0.2),
			inset 0 -4px 15px rgba(0, 0, 0, 0.1);

		/* 动画效果 */
		animation:
			rainbow 3s infinite ease-in-out,
			float 6s infinite ease-in-out;
		/* 新增漂浮效果 */
		will-change: transform, color;
		/* 性能优化 */
		transform: translateZ(0);
		/* 触发GPU加速 */

		&::after {
			content: '✦ ✧ ✦';
			position: absolute;
			top: -40px;
			/* 扩大间距 */
			left: 0;
			width: 100%;
			text-align: center;
			font-size: 2.2rem;
			letter-spacing: 20px;
			/* 增加间距避免重叠 */
			text-indent: 20px;
			/* 偏移量修正 */
			opacity: 0.9;
			text-shadow:
				0 0 15px rgba(255, 215, 255, 0.9),
				0 0 30px rgba(255, 100, 255, 0.6);
			/* 双色闪光 */
			animation: sparkle 4s infinite alternate;
		}


		/* 彩虹动画优化 */
		@keyframes rainbow {

			0%,
			100% {
				color: #ff7eb3;
				/* 更鲜艳的粉色 */
				text-shadow-color: rgba(255, 215, 255, 0.9);
				/* 同步变化的文字阴影 */
			}

			25% {
				color: #ffcc00;
				text-shadow-color: rgba(255, 230, 100, 0.9);
			}

			50% {
				color: #7afcff;
				text-shadow-color: rgba(130, 255, 255, 0.9);
			}

			75% {
				color: #a4f8db;
				text-shadow-color: rgba(180, 255, 220, 0.9);
			}
		}

		/* 新增星星闪烁动画 */
		@keyframes sparkle {
			0% {
				opacity: 0.5;
				transform: scale(0.95);
			}

			100% {
				opacity: 1;
				transform: scale(1.15);
				text-shadow:
					0 0 25px rgba(255, 220, 255, 1),
					0 0 40px rgba(255, 50, 255, 0.8);
			}
		}

		/* 新增漂浮效果 */
		@keyframes float {

			0%,
			100% {
				transform: translateY(0) rotate(0.5deg);
			}

			50% {
				transform: translateY(-15px) rotate(-0.5deg);
			}
		}
	}

	.restart-btn {
		background: linear-gradient(to right, #6a93cb, #a4bfef);
		color: white;
		border: none;
		padding: 15rpx 40rpx;
		font-size: 20px;
		border-radius: 30px;
		cursor: pointer;
		transition: all 0.3s;
		font-weight: bold;
		box-shadow: 0 5px 20px rgba(106, 147, 203, 0.5);
	}

	.restart-btn:hover {
		transform: translateY(-3px);
		box-shadow: 0 8px 25px rgba(106, 147, 203, 0.7);
	}


	.scaleAnimation {
		/* 应用动画 */
		animation: scalePulse 1.5s ease-in-out infinite;
		/* 缩放中心点 */
		transform-origin: center;
	}

	/* 添加缩放动画关键帧 */
	@keyframes scalePulse {
		0% {
			transform: scale(1);
		}

		50% {
			transform: scale(1.2);
		}

		100% {
			transform: scale(1);
		}
	}
	
	.glowPulse {
		animation: glowPulse 2s infinite;
		border-radius: 50%;
	}
	
	.pulse {
		animation: pulse 2s infinite;
		border-radius: 50%;
	}
	
	@keyframes pulse {
		0% {
			box-shadow: 0 0 0 0 rgba(255, 255, 255, 0.7);
		}
	
		70% {
			box-shadow: 0 0 0 12px rgba(255, 255, 255, 0);
			
		}
	
		100% {
			box-shadow: 0 0 0 0 rgba(255, 255, 255, 0);
		}
	}
	
	@keyframes glowPulse {
	
		0%,
		100% {
			opacity: 0.7;
			
		}
	
		50% {
			opacity: 0.9;
			transform: scale(1.2);
		}
	}
</style>