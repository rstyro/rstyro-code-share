<template>
	<view>
		<!-- 设置弹窗 -->
		<view class="modal" :class="{show}">

			<view class="modal-content">
				<view class="close-btn" @click="closeModal">
					<cl-icon type="icon-guanbi" size="36" color="#aaa" />
				</view>
				<view class="modal-title">设置</view>

				<view class="setting-group">
					<view class="setting-title">版本设置</view>
					<view class="difficulty-options">
						<view v-for="item in versionOptions" :key="item.value" class="difficulty-btn"
							:class="{ active: version === item.value }" @click="version = item.value">
							<view v-if="item.hot" class="hot-tag">{{item.hotText}}</view>
							{{ item.label }}
						</view>
					</view>
				</view>

				<view class="action-btns">
					<button v-if="version!='normal'" class="btn edit-btn" @click="editSettings">
						<!-- <cl-icon class="icon" type="icon-ad" size="20" color="#fff" /> -->
						编辑</button>
					<button class="btn apply-btn" @click="applySettings">
						<!-- <cl-icon v-if="version!='normal'" class="icon" type="icon-ad" size="20" color="#fff" /> -->
							应用设置</button>
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
			options: {
				type: Array,
				require: false
			}
		},
		computed: {
			versionOptions() {
				if (this.options && this.options.length > 0) {
					return this.options;
				}
				return [];
			}
		},
		data() {
			return {
				version: 'normal',
				isAdLoading: false,
				cmd: '',
				callbackData: {
					userId: '',
					extra: ''
				}
			};
		},
		onReady() {
			
		},
		methods: {
			closeModal() {
				this.$emit('close')
			},
			// 应用设置
			applySettings() {
				this.cmd = 'apply';
				this.viewAd();
			},
			editSettings() {
				this.cmd = 'edit';
				this.viewAd();
			},
			viewAd() {
				this.callbackData.userId = this.store_uid || '0';
				this.callbackData.extra = this.store_token || 'null';
				this.$emit(this.cmd, this.version);
				this.closeModal();
			}
		}
	}
</script>

<style lang="scss" scoped>
	/* 弹窗样式 */
	.modal {
		position: fixed;
		top: 10px;
		left: 0;
		width: 100%;
		height: 100%;
		background: rgba(0, 0, 0, 0.7);
		display: flex;
		justify-content: center;
		align-items: center;
		z-index: 100;
		opacity: 0;
		pointer-events: none;
		transition: opacity 0.1s;
		overflow: auto;
	}

	.modal.show {
		opacity: 1;
		pointer-events: all;
	}

	.modal-content {
		background: white;
		border-radius: 25px;
		width: 90%;
		max-height: 80vh;
		overflow-y: auto;
		padding: 40px 30px;
		text-align: center;
		position: relative;
		transform: translateY(30px);
		transition: transform 0.4s;
		box-shadow: 0 25px 50px rgba(0, 0, 0, 0.3);
	}

	.modal.show .modal-content {
		transform: translateY(0);
	}

	.modal-title {
		font-size: 28px;
		padding: 12px;
		margin: 20px auto;
		border-radius: 15px;
		color: white;
		font-weight: bold;
		background: linear-gradient(to right, #ff7eb3, #ff758c);
		position: relative;
	}

	.close-btn {
		position: absolute;
		padding: 20rpx;
		top: 10rpx;
		right: 15px;
		border: none;
		font-size: 24px;
		color: #666;
		cursor: pointer;
		transition: all 0.3s;
		z-index: 9;
	}

	.close-btn:hover {
		color: #ff758c;
		transform: rotate(90deg);
	}

	/* 设置面板样式 */
	.setting-group {
		margin: 20px 0;
		padding: 15px;
		background: #fff9fc;
		border-radius: 15px;
		border: 2px dashed #ffd6e7;
	}

	.setting-title {
		font-size: 18px;
		margin-bottom: 15px;
		color: #ff6b9c;
		display: flex;
		align-items: center;
		gap: 10px;
	}


	.difficulty-options {
		display: flex;
		flex-direction: column;
		gap: 10px;
		margin-top: 10px;
	}

	.difficulty-btn {
		position: relative;
		flex: 1;
		padding: 12px;
		background: #f0f0f0;
		border: 2px solid #e0e0e0;
		border-radius: 10px;
		cursor: pointer;
		font-weight: bold;
		color: #666;
		transition: all 0.3s;
		overflow: hidden;
	}

	.difficulty-btn.active {
		background: #ff7eb3;
		color: white;
		border-color: #ff7eb3;
		box-shadow: 0 4px 10px rgba(255, 126, 179, 0.3);
	}

	/* HOT 标签样式 */
	.hot-tag {
		position: absolute;
		top: 0rpx;
		left: -20rpx;
		background: #ff4d4f;
		color: #fff;
		font-size: 20rpx;
		padding: 5rpx 20rpx;
		border-radius: 5px;
		white-space: nowrap;
		z-index: 1;
		transform: rotate(-45deg);
		box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
	}

	.action-btns {
		display: flex;
		justify-items: center;
		align-items: center;
		gap: 20rpx;

		.btn {
			color: white;
			flex: 1;
			border: none;
			padding: 15px 30rpx;
			font-size: 18px;
			border-radius: 30px;
			cursor: pointer;
			transition: all 0.3s;
			font-weight: bold;
			margin-top: 20px;
			box-shadow: 0 8px 20px rgba(255, 117, 140, 0.4);

			&:hover {
				transform: translateY(-3px);
				box-shadow: 0 12px 25px rgba(255, 117, 140, 0.5);
			}
		}

		.edit-btn {
			background: linear-gradient(to right, #6a93cb, #a4bfef);
		}

		.apply-btn {
			background: linear-gradient(to right, #ff7eb3, #ff758c);
		}

		.icon {
			margin-right: 10rpx;
		}

	}


	.ad-error {
		color: orangered;
		margin-top: 5px;
	}
</style>