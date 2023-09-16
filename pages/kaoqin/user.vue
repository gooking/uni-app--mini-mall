<template>
	<!-- <view class="page">
		<page-box-empty v-if="!workingHoursMySubmitLogs || workingHoursMySubmitLogs.length == 0" title="暂无工时记录" sub-title="立即提交今日工时试试吧～" :show-btn="false" />
		<uni-list v-else>
			<uni-list-item v-for="(item,index) in workingHoursMySubmitLogs" :key="item.id" :title="item.day" :note="item.projectName" :rightText="item.number + '小时'"></uni-list-item>
		</uni-list>
		<view class="btn-box">
			<button type="primary" @click="_submit">上报工时</button>
		</view>
	</view> -->
	<view class="page2">
		<view v-if="todayDone" class="main">
			<uni-icons type="checkbox-filled" size="200rpx" color="#18bc37"></uni-icons>
			<view class="txt">{{ sysconfigMap.todayTip1 }}</view>
		</view>
		<view v-else class="main">
			<uni-icons type="loop" size="200rpx" color="#ccc"></uni-icons>
			<view class="txt">{{ sysconfigMap.todayTip0 }}</view>
		</view>
		<view class="btm">
			<button type="primary" @click="_submit">上报工时</button>
		</view>
	</view>
</template>

<script>
	const dayjs = require('dayjs')
	export default {
		data() {
			return {
				page: 1,
				workingHoursMySubmitLogs: undefined,
				todayDone: false,
			}
		},
		onLoad(e) {
			this._workingHoursMySubmitLogs()
			// // 读取二维码中的考勤ID
			// if (e && e.workingHoursProjectId) {
			//   this.$u.vuex('workingHoursProjectId', e.workingHoursProjectId)
			// }
			// // #ifdef MP-WEIXIN
			// if (e && e.scene) {
			//   const scene = decodeURIComponent(e.scene)
			//   if (scene) {
			// 	this.$u.vuex('workingHoursProjectId', scene)
			//   }
			// }
			// // #endif
		},
		onShow() {
			const needRefresh = uni.getStorageSync('needRefresh')
			if (needRefresh) {
				this.page = 1
				this._workingHoursMySubmitLogs()
				uni.removeStorageSync('needRefresh')
			}
		},
		// onReachBottom() {
		// 	this.page++;
		// 	this._workingHoursMySubmitLogs()
		// },
		methods: {
			async _workingHoursMySubmitLogs() {
				uni.showLoading({
					title: ''
				})
				const res = await this.$api.workingHoursMySubmitLogs({
					token: this.token,
					page: this.page,
					dateAddBegin: dayjs().format('YYYY-MM-DD')
				})
				uni.hideLoading()
				// if (res.code == 0) {
				// 	res.data.result.forEach(ele => {
				// 		const project = res.data.projectMap[ele.projectId]
				// 		if (project) {
				// 			ele.projectName = project.name
				// 		}
				// 	})
				// 	if (this.page == 1) {
				// 		this.workingHoursMySubmitLogs = res.data.result
				// 	} else {
				// 		this.workingHoursMySubmitLogs = this.workingHoursMySubmitLogs.concat(res.data.result)
				// 	}
				// } else {
				// 	if (this.page == 1) {
				// 		this.workingHoursMySubmitLogs = null
				// 	} else {
				// 		uni.showToast({
				// 			title: '没有更多了~',
				// 			icon: 'none'
				// 		})
				// 	}
				// }
				if (res.code == 0) {
					this.todayDone = true
				} else {
					this.todayDone = false
				}
			},
			_submit() {
				uni.showModal({
					title: '请填写公司编号',
					content: '',
					editable: true,
					placeholderText: '输入公司编号',
					success: (res) => {
						if (!res.confirm) {
							return
						}
						if (!res.content) {
							uni.showToast({
								title: '请填写公司编号',
								icon: 'none'
							})
							return
						}
						this.submit(res.content)
					}
				})
			},
			async submit(code) {
				const res = await this.$api.workingHoursProjectInfo(code)
				if(res.code != 0) {
					uni.showModal({
						content: res.msg,
						showCancel: false
					})
					return
				}
				uni.navigateTo({
					url: '/pages/kaoqin/submit?id=' + res.data.id
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.page {
		padding-bottom: 110rpx;
	}
	.btn-box {
		z-index: 2;
		position: fixed;
		bottom: 0;
		left: 0;
		width: 100vw;
		background-color: #fff;
	}
	.page2 {
		width: 100vw;
		height: 100vh;
		display: flex;
		flex-direction: column;
		.main {
			flex: 1;
			display: flex;
			flex-direction: column;
			justify-content: center;
			align-items: center;
			.txt {
				font-size: 28rpx;
				color: #333;
				margin-top: 32rpx;
			}
		}
	}
</style>
