<template>
	<view v-if="userDetail">
		<!-- #ifdef MP-WEIXIN || MP-BAIDU || MP-QQ -->
		<view class="blank-top"></view>
		<view class="u-flex user-box u-p-l-30 u-p-r-20 u-p-b-30">
			<view class="u-m-r-10">
				<open-data type="userAvatarUrl"></open-data>
			</view>
			<view class="u-flex-1">
				<view class="u-font-16 u-p-b-20"><open-data type="userNickName"></open-data></view>
				<view v-if="userDetail" class="u-font-14 u-tips-color">用户编号:{{ userDetail.base.id }}</view>
			</view>
		</view>
		<!-- #endif -->
		<!-- #ifndef MP-WEIXIN || MP-BAIDU || MP-QQ -->
		<view class="u-flex user-box u-p-l-30 u-p-r-20 u-p-b-30" style="padding-top: 32rpx;">
			<view class="u-m-r-10">
				<image class="avatar" :src="userDetail.base.avatarUrl" mode="aspectFill"></image>
			</view>
			<view class="u-flex-1">
				<view class="u-font-16 u-p-b-20"><open-data type="userNickName"></open-data></view>
				<view v-if="userDetail" class="u-font-14 u-tips-color">用户编号:{{ userDetail.base.id }}</view>
			</view>
		</view>
		<!-- #endif -->
		<u-grid :col="3">
			<u-grid-item @click="go('/pages/order/index?status=0')">
				<u-icon name="red-packet" :size="46" color="#a78845"></u-icon>
				<view class="grid-text">待支付</view>
			</u-grid-item>
			<u-grid-item @click="go('/pages/order/index?status=1')">
				<u-icon name="car" :size="46" color="#a78845"></u-icon>
				<view class="grid-text">待发货</view>
			</u-grid-item>
			<u-grid-item @click="go('/pages/order/index?status=2')">
				<u-icon name="gift" :size="46" color="#a78845"></u-icon>
				<view class="grid-text">已发货</view>
			</u-grid-item>
		</u-grid>
		<view class="u-m-t-20">
			<u-cell-group>
				<u-cell-item icon="heart" title="我的收藏" @click="go('/pages/goods/fav')"></u-cell-item>
				<u-cell-item icon="map" title="收货地址" @click="go('/pages/address/index')"></u-cell-item>
				<u-cell-item icon="server-man" title="关于我们" @click="go('/pages/about/about?key=aboutus')"></u-cell-item>
			</u-cell-group>
		</view>
		<view class="version">version {{ version }}</view>
	</view>
	<view v-else>
		<page-box-empty title="请先登陆" sub-title="登陆以后才能访问用户中心" :show-btn="true" btn-title="立即登陆" url="/pages/login/login" />
	</view>
</template>

<script>
	export default {
		data() {
			return {
				userDetail: undefined,
				version: getApp().globalData.version
			}
		},
		onShow() {
			getApp().checkHasLoginedH5().then(isLogin => {
				if(isLogin) {
					this._userDetail()
				}
			})
		},
		methods: {
			async _userDetail() {
				const res = await this.$api.userDetail(this.token)
				if(res.code == 0) {
					if (!res.data.base.avatarUrl) {
						res.data.base.avatarUrl = '/static/images/empty.jpg'
					}
					this.userDetail = res.data
				}
			},
			go(url) {
				uni.navigateTo({
					url: url
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
page{
	background-color: #ededed;
}
.blank-top {
	height: 88rpx;
}
.grid-text {
	color: #a78845;
}

.camera{
	width: 54px;
	height: 44px;
	
	&:active{
		background-color: #ededed;
	}
}
.user-box{
	background-color: #fff;
}
.version {
	margin-top: 64rpx;
	text-align: center;
	color: #999999;
}
.avatar {
		width: 128rpx;
		height: 128rpx;
		border-radius: 50%;
	}
</style>
