<template>
	<view>
		<view class="wrap">
			<u-sticky>
				<view class="u-tabs-box">
					<u-tabs activeColor="#f29100" ref="tabs" :list="list" :current="current" @change="change" :is-scroll="false" swiperWidth="750"></u-tabs>
				</view>
			</u-sticky>
			<page-box-empty v-if="!orderList || orderList.length == 0" title="您还没有相关的订单" sub-title="可以去看看有那些想买的～" :show-btn="true" />
			<view v-else class="page-box">
				<view class="order" v-for="(item, index) in orderList" :key="item.id">
					<view class="top" @click="godetail(item.id)">
						<view class="left">
							订单号: {{ item.orderNumber }}
						</view>
						<view class="right">{{ item.statusStr }}</view>
					</view>
					<view class="item" v-for="(item2, index2) in item.goodsList" :key="item2.id">
						<view class="left"><image :src="item2.pic" mode="aspectFill"></image></view>
						<view class="content">
							<view class="title u-line-2">{{ item2.goodsName }}</view>
							<view v-if="item2.property" class="type">{{ item2.property }}</view>
						</view>
						<view class="right">
							<view class="price">
								￥{{ item2.amountSingle }}
							</view>
							<view class="number">x{{ item2.number }}</view>
						</view>
					</view>
					<view class="total">
						共{{ item.goodsNumber }}件商品 合计:
						<text class="total-price">
							￥{{ item.amountReal }}
						</text>
					</view>
					<view v-if="item.status == 0" class="bottom">
						<view class="exchange btn" @click="close(item.id)">取消订单</view>
						<view class="evaluate btn u-margin-left-24" @click="pay(index)">立即支付</view>
					</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
const wxpay = require('@/common/wxpay.js')
export default {
	data() {
		return {
			orderList: undefined,
			dataList: undefined,
			list: [
				{
					name: '待付款',
					count: 0
				},
				{
					name: '待发货'
				},
				{
					name: '待收货'
				},
			],
			current: 0,
			tabsHeight: 0,
		};
	},
	onLoad(e) {
		if(!e.status) {
			e.status = 0
		}
		this.current = e.status
		this.change(e.status)
	},
	computed: {
		// 价格小数
		priceDecimal() {
			return val => {
				if (val !== parseInt(val)) return val.slice(-2);
				else return '00';
			};
		},
		// 价格整数
		priceInt() {
			return val => {
				if (val !== parseInt(val)) return val.split('.')[0];
				else return val;
			};
		}
	},
	methods: {
		async _orderStatistics(){
			const res = await this.$api.orderStatistics(this.token)
			if(res.code == 0) {
				this.list[0].count = res.data.count_id_no_pay
				this.list[1].count = res.data.count_id_no_transfer
				this.list[2].count = res.data.count_id_no_confirm
				this.list.splice(0, 0)
			}
		},
		// 页面数据
		async getOrderList() {
			this.orderList = null
			const res = await this.$api.orderList({
				token: this.token,
				status: this.current
			})
			if(res.code == 0) {
				const goodsMap = res.data.goodsMap
				res.data.orderList.forEach(ele => {
					ele.goodsList = goodsMap[ele.id]
				})
				this.orderList = res.data.orderList
			}
		},
		goIndex() {
			uni.switchTab({
				url: "../index/index"
			})
		},
		// tab栏切换
		change(index) {
			this.current = index
			this._orderStatistics()
			this.getOrderList();
		},
		async close(orderId) {
			uni.showModal({
			    title: '请确认',
			    content: '确定要取消该订单吗？',
			    success: res => {
			        if (res.confirm) {
			            this._close(orderId)
			        }
			    }
			});
		},
		async _close(orderId) {
			const res = await this.$api.orderClose(this.token, orderId)
			if(res.code != 0) {
				uni.showToast({
					title: res.msg,
					icon: 'none'
				})
			} else {
				uni.showToast({
					title: '已取消'
				})
				this.change(this.current)
			}
		},
		async pay(index) {
			let res = await this.$api.userAmount(this.token)
			let balance = 0
			if(res.code != 0) {
				uni.showToast({
					title: res.msg,
					icon: 'none'
				})
			}
			balance = res.data.balance
			const orderInfo = this.orderList[index]
			const needPay = (orderInfo.amountReal - balance).toFixed(2)
			if(needPay <= 0) {
				// 直接调用支付接口
				const res = await this.$api.orderPay(this.token, orderInfo.id)
				if(res.code != 0) {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
				} else {
					uni.showToast({
						title: '支付成功'
					})
					this.change(1)
				}
			} else {
				// 发起在线支付
				// #ifdef MP-WEIXIN
				wxpay.wxpay('order', needPay, orderInfo.id, '')
				// #endif
				// #ifndef MP-WEIXIN
				uni.showModal({
				    title: '提示',
				    content: '支付功能即将上线，请暂时联系客服人员进行线下支付',
					showCancel: false,
					confirmText: '我知道了'
				})
				// #endif
			}
		},
		godetail(orderId) {
			uni.navigateTo({
				url: './detail?id=' + orderId
			})
		},
	}
};
</script>

<style>
/* #ifndef H5 */
page {
	background-color: #f2f2f2;
}
/* #endif */
</style>

<style lang="scss" scoped>
.order {
	width: 710rpx;
	background-color: #ffffff;
	margin: 20rpx auto;
	border-radius: 20rpx;
	box-sizing: border-box;
	padding: 20rpx;
	font-size: 28rpx;
	.top {
		display: flex;
		justify-content: space-between;
		.left {
			display: flex;
			align-items: center;
			.store {
				margin: 0 10rpx;
				font-size: 32rpx;
				font-weight: bold;
			}
		}
		.right {
			color: $u-type-warning-dark;
		}
	}
	.item {
		display: flex;
		margin: 20rpx 0 0;
		.left {
			margin-right: 20rpx;
			image {
				width: 200rpx;
				height: 200rpx;
				border-radius: 10rpx;
			}
		}
		.content {
			.title {
				font-size: 28rpx;
				line-height: 50rpx;
			}
			.type {
				margin: 10rpx 0;
				font-size: 24rpx;
				color: $u-tips-color;
			}
			.delivery-time {
				color: #e5d001;
				font-size: 24rpx;
			}
		}
		.right {
			margin-left: 10rpx;
			padding-top: 20rpx;
			text-align: right;
			.decimal {
				font-size: 24rpx;
				margin-top: 4rpx;
			}
			.number {
				color: $u-tips-color;
				font-size: 24rpx;
			}
		}
	}
	.total {
		margin-top: 20rpx;
		text-align: right;
		font-size: 24rpx;
		.total-price {
			font-size: 32rpx;
		}
	}
	.bottom {
		display: flex;
		margin-top: 40rpx;
		padding: 0 10rpx;
		justify-content: end;
		align-items: center;
		.btn {
			line-height: 52rpx;
			width: 160rpx;
			border-radius: 26rpx;
			border: 2rpx solid $u-border-color;
			font-size: 26rpx;
			text-align: center;
			color: $u-type-info-dark;
		}
		.evaluate {
			color: $u-type-warning-dark;
			border-color: $u-type-warning-dark;
		}
	}
}
.wrap {
	display: flex;
	flex-direction: column;
	height: calc(100vh - var(--window-top));
	width: 100%;
}
.swiper-box {
	flex: 1;
}
.swiper-item {
	height: 100%;
}
</style>
