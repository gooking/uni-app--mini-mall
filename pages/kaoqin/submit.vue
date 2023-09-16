<template>
	<view class="page">
		<view class="title">{{ today }}</view>
		<view class="sub-title">请选择今日工时:</view>
		<uni-list>
			<uni-list-item v-for="(item,index) in 30" :title="0.5*(index+1) + '小时'" rightText="立即上报" clickable showArrow @click="submit(index)"></uni-list-item>
		</uni-list>
	</view>
</template>

<script>
	const dayjs = require('dayjs')
	export default {
		data() {
			return {
				active: 4,
				today: dayjs().format('YYYY-MM-DD')
			}
		},
		onLoad(e) {
			this.id = e.id
		},
		onShow() {
			
		},
		methods: {
			async submit(index) {
				const number = 0.5*(index+1)
				uni.showLoading({
					title: ''
				})
				const res = await this.$api.workingHoursSubmit({
					token: this.token,
					projectId: this.id,
					number
				})
				uni.hideLoading()
				if (res.code == 0) {
					uni.showModal({
						content: '上报成功',
						showCancel:false,
						success: () => {
							uni.setStorageSync('needRefresh', true)
							uni.navigateBack()
						}
					})
				} else {
					uni.showModal({
						content: res.msg,
						showCancel:false
					})
				}
			},
		}
	}
</script>

<style lang="scss" scoped>
	.btn-box {
		z-index: 2;
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100vw;
		background-color: #fff;
	}
	.title {
		text-align: center;
		font-weight: bold;
		padding: 16rpx;
	}
	.sub-title {
		padding: 16rpx;
		color: #999;
		font-size: 26rpx;
	}
	.grid {
		padding: 32rpx;
		.item {
			display: flex;
			justify-content: center;
			align-items: center;
			flex: 1;
			font-size: 24rpx;
			text {
				color: #f76260;
				font-size: 34rpx;
				font-weight: bold;
				padding-right: 6rpx;
			}
		}
		.active {
			background-color: #09bb07;
			color: #fff;
			text {
				color: #fff;
			}
		}
	}
</style>
