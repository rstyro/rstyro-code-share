<template>
	<view class="task-popup" :class="{show}">
		<view class="task-content">
			<view class="task-title " :class="{woman:!isMan,man:isMan}">
				<text v-if="isMan">男生做任务</text>
				<text v-else>女生做任务</text>
			</view>
			<view class="task-text">{{ taskContent }}</view>
			<view class="task-reward">
				<view class="success">完成奖励：执行者随机前进1-3格</view>
				<view class="fail">未完成惩罚：执行者后退前进3-6格</view>
			</view>
			<view class="action-btns">
				<button class="btn complete-btn"  @click="completeTask">
					<text>已完成</text>
				</button>
				<button class="btn uncomplete-btn" @click="closeTask" >未完成</button>
			</view>
			
		</view>
	</view>
	</view>
	</view>

</template>

<script>
	export default {
		props: {
			show: {
				type: Boolean,
				required: true,
				default: false
			},
			isMan: {
				type: Boolean,
				required: true,
				default: false
			},
			taskContent: {
				type: String,
				required: true,
				default: '未知任务'
			}
		},
		data() {
			return {
				isButtonEnabled: false,
				countdown: 1,
				timer: null
			}
		},
		watch: {
			show(newVal) {
			// 	if (newVal) {
			// 		this.isButtonEnabled = false;
			// 		this.countdown = 1;
			// 		this.timer = setInterval(() => {
			// 			if (this.countdown > 0) {
			// 				this.countdown--;
			// 			} else {
			// 				clearInterval(this.timer);
			// 				this.isButtonEnabled = true;
			// 			}
			// 		}, 1000);
			// 	} else {
			// 		if (this.timer) {
			// 			clearInterval(this.timer);
			// 			this.timer = null;
			// 		}
			// 	}
			}
		},
		methods: {
			closeTask() {
				// if (this.timer) {
				// 	clearInterval(this.timer);
				// 	this.timer = null;
				// }
				this.$emit('close')
			},
			completeTask(){
				this.$emit('complete')
			}
		}
	}
</script>

<style lang="scss" scoped>
/* 优化弹窗样式 */
.task-popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000; /* 提升层级确保覆盖 */
  opacity: 0;
  pointer-events: none;
  transition: opacity 0.3s ease-out; /* 平滑淡入淡出 */
}

.task-popup.show {
  opacity: 1;
  pointer-events: all;
}

.task-content {
  background: white;
  border-radius: 25rpx;
  width: 85%;
  padding: 40rpx;
  text-align: center;
  position: relative;
  transform: translateY(20px) scale(0.9); /* 增加缩放动画 */
  transition: transform 0.4s cubic-bezier(0.18, 0.89, 0.32, 1.28); /* 弹性动画 */
  box-shadow: 0 25rpx 50rpx rgba(0, 0, 0, 0.3);
  overflow: hidden;
  
  /* 添加顶部装饰条 */
  &::before {
    content: '';
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    height: 8rpx;
    background: linear-gradient(90deg, #ff3366, #ff9966);
    border-radius: 4rpx 4rpx 0 0;
  }
}

.task-popup.show .task-content {
  transform: translateY(0) scale(1); /* 弹出动画 */
}

/* 优化标题样式 */
.task-title {
  font-size: 40rpx;
  margin-bottom: 40rpx;
  padding: 30rpx 20rpx;
  border-radius: 20rpx;
  color: white;
  font-weight: bold;
  position: relative;
  letter-spacing: 3rpx;
  text-shadow: 0 2rpx 4rpx rgba(0,0,0,0.2);
  transition: all 0.3s;
  
  /* 添加图标装饰 */
  &::after, &::before {
    content: "★";
    position: absolute;
    top: 50%;
    transform: translateY(-50%);
    font-size: 30rpx;
    opacity: 0.8;
  }
  
  &::before { left: 40rpx; }
  &::after { right: 40rpx; }
}

.man {
  background: linear-gradient(to right, #4776E6, #8E54E9);
  box-shadow: 0 10rpx 20rpx rgba(106, 147, 203, 0.4);
}

.woman {
  background: linear-gradient(to right, #FF5E7D, #FF8A5C);
  box-shadow: 0 10rpx 20rpx rgba(255, 154, 158, 0.4);
}

/* 优化任务文本样式 */
.task-text {
  font-size: 40rpx;
  margin: 50rpx 0;
  color: #ff3366;
  line-height: 1.6;
  min-height: 160rpx;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #fff9fc;
  border-radius: 20rpx;
  padding: 40rpx;
  border: 2rpx dashed #ffd6e7;
  box-shadow: inset 0 0 20rpx rgba(255,182,193,0.1);
  font-weight: 500;
  transition: transform 0.3s;
  
  /* 任务文本悬浮效果 */
  &:hover {
    transform: translateY(-5rpx);
  }
}

/* 优化奖励区域样式 */
.task-reward {
  display: flex;
  flex-direction: column;
  margin: 50rpx 0;
  font-size: 32rpx;
  text-align: left;
  
  > view {
    padding: 20rpx 40rpx;
    border-radius: 15rpx;
    margin-bottom: 25rpx;
    position: relative;
    padding-left: 70rpx;
    
    &::before {
      content: '';
      position: absolute;
      left: 30rpx;
      top: 50%;
      transform: translateY(-50%);
      width: 36rpx;
      height: 36rpx;
      border-radius: 50%;
      background: white;
    }
  }
  
  .success {
    background: rgba(46, 204, 113, 0.15);
    color: #27ae60;
    
    &::before {
      content: '✓';
      display: flex;
      align-items: center;
      justify-content: center;
      background: #2ecc71;
      color: white;
    }
  }
  
  .fail {
    background: rgba(231, 76, 60, 0.15);
    color: #c0392b;
    
    &::before {
      content: '!';
      display: flex;
      align-items: center;
      justify-content: center;
      background: #e74c3c;
      color: white;
    }
  }
}

/* 优化按钮区域 */
.action-btns {
  display: flex;
  justify-content: space-around;
  gap: 40rpx; /* 增加按钮间距 */
  margin-top: 30rpx;
  
  .btn {
    flex: 1;
    color: white;
    border: none;
    padding: 25rpx 20rpx;
    font-size: 34rpx;
    border-radius: 50rpx;
    cursor: pointer;
    transition: all 0.3s ease;
    font-weight: bold;
    box-shadow: 0 8rpx 20rpx rgba(0, 0, 0, 0.2);
    position: relative;
    overflow: hidden;
    letter-spacing: 2rpx;
    
    &::after {
      content: '';
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      background: rgba(255, 255, 255, 0.2);
      opacity: 0;
      transition: opacity 0.3s;
    }
    
    &:active::after {
      opacity: 1;
    }
  }
  
  .uncomplete-btn {
    background: linear-gradient(135deg, #FF416C, #FF4B2B);
    &:hover {
      transform: translateY(-6rpx);
      box-shadow: 0 12rpx 25rpx rgba(255, 75, 43, 0.4);
    }
  }
  
  .complete-btn {
    background: linear-gradient(135deg, #38ef7d, #11998e);
    &:hover {
      transform: translateY(-6rpx);
      box-shadow: 0 12rpx 25rpx rgba(17, 153, 142, 0.4);
    }
  }
  
  /* 按钮点击效果 */
  .btn:active {
    transform: scale(0.96);
  }
}

</style>