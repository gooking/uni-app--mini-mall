<template>
	<view class="u-wrap">
		<view class="u-menu-wrap">
			<scroll-view scroll-y scroll-with-animation class="u-tab-view menu-scroll-view" :scroll-top="scrollTop">
				<view v-for="(item,index) in tabbar" :key="index" class="u-tab-item" :class="[current==index ? 'u-tab-item-active' : '']"
				 :data-current="index" @tap.stop="swichMenu(index)">
					<text class="u-line-1">{{item.name}}</text>
				</view>
			</scroll-view>
			<scroll-view scroll-y class="right-box">
				<view class="page-view">
					<view class="class-item">
						<page-box-empty v-if="!goodsList || goodsList.length == 0" title="暂无商品" sub-title="到别的分类看看吧～" :show-btn="false" />
						<view class="item-container">
							<view class="thumb-box" v-for="(item1, index1) in goodsList" :key="index1">
								<image class="item-menu-image" :src="item1.pic" mode="aspectFill" @click="goGoods(item1.id)"></image>
								<view class="item-menu-name">
									<view class="u-line-2" @click="goGoods(item1.id)">{{item1.name}}</view>
								</view>
							</view>
						</view>
					</view>
				</view>
			</scroll-view>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				tabbar: undefined,
				scrollTop: 0, //tab标题的滚动条位置
				current: 0, // 预设当前项的值
				menuHeight: 0, // 左边菜单的高度
				menuItemHeight: 0, // 左边菜单item的高度
				goodsList: undefined, // 商品列表
			}
		},
		onLoad() {
			this._goodsCategory()
		},
		methods: {
			async _goodsCategory() {
				const res = await this.$api.goodsCategory()
				if(res.code == 0) {
					this.tabbar = res.data
					this.tabbar.splice(0, 0, {
						id: 0,
						name: '全部',
					})
					this.fetchGoods(0)
				}
			},
			async fetchGoods(categoryIndex) {
				const item = this.tabbar[categoryIndex]
				const res = await this.$api.goodsv2({
					token: this.token,
					categoryId: item.id ? item.id : ''
				})
				this.goodsList = null
				if (res.code != 0 && res.code != 700) {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
				} else if(res.code == 0) {
					this.goodsList = res.data.result
				}
			},
			// 点击左边的栏目切换
			async swichMenu(index) {
				if(index == this.current) return ;
				this.current = index;
				// 如果为0，意味着尚未初始化
				if(this.menuHeight == 0 || this.menuItemHeight == 0) {
					await this.getElRect('menu-scroll-view', 'menuHeight');
					await this.getElRect('u-tab-item', 'menuItemHeight');
				}
				// 将菜单菜单活动item垂直居中
				this.scrollTop = index * this.menuItemHeight + this.menuItemHeight / 2 - this.menuHeight / 2;
				this.fetchGoods(index)
			},
			// 获取一个目标元素的高度
			getElRect(elClass, dataVal) {
				new Promise((resolve, reject) => {
					const query = uni.createSelectorQuery().in(this);
					query.select('.' + elClass).fields({size: true},res => {
						// 如果节点尚未生成，res值为null，循环调用执行
						if(!res) {
							setTimeout(() => {
								this.getElRect(elClass);
							}, 10);
							return ;
						}
						this[dataVal] = res.height;
					}).exec();
				})
			},
			goGoods(id) {
				console.log(id);
				uni.navigateTo({
					url: '../goods/details?id=' + id
				})
			},
		}
	}
</script>

<style lang="scss" scoped>
	.u-wrap {
		height: calc(100vh);
		/* #ifdef H5 */
		height: calc(100vh - var(--window-top));
		/* #endif */
		display: flex;
		flex-direction: column;
	}

	.u-search-box {
		padding: 18rpx 30rpx;
	}

	.u-menu-wrap {
		flex: 1;
		display: flex;
		overflow: hidden;
	}

	.u-search-inner {
		background-color: rgb(234, 234, 234);
		border-radius: 100rpx;
		display: flex;
		align-items: center;
		padding: 10rpx 16rpx;
	}

	.u-search-text {
		font-size: 26rpx;
		color: $u-tips-color;
		margin-left: 10rpx;
	}

	.u-tab-view {
		width: 200rpx;
		height: 100%;
	}

	.u-tab-item {
		height: 110rpx;
		background: #f6f6f6;
		box-sizing: border-box;
		display: flex;
		align-items: center;
		justify-content: center;
		font-size: 26rpx;
		color: #444;
		font-weight: 400;
		line-height: 1;
	}
	
	.u-tab-item-active {
		position: relative;
		color: #000;
		font-size: 30rpx;
		font-weight: 600;
		background: #fff;
	}
	
	.u-tab-item-active::before {
		content: "";
		position: absolute;
		border-left: 4px solid $u-type-primary;
		height: 32rpx;
		left: 0;
		top: 39rpx;
	}

	.u-tab-view {
		height: 100%;
	}
	
	.right-box {
		background-color: rgb(250, 250, 250);
	}
	
	.page-view {
		padding: 16rpx;
	}
	
	.class-item {
		margin-bottom: 30rpx;
		background-color: #fff;
		padding: 16rpx;
		border-radius: 8rpx;
		.empty {
			padding-top: 200rpx;
		}
	}
	
	.item-title {
		font-size: 26rpx;
		color: $u-main-color;
		font-weight: bold;
	}
	
	.item-menu-name {
		font-weight: normal;
		font-size: 24rpx;
		color: $u-main-color;
		padding: 8rpx;
	}
	
	.item-container {
		display: flex;
		flex-wrap: wrap;
	}
	
	.thumb-box {
		width: 50%;
		display: flex;
		align-items: center;
		justify-content: center;
		flex-direction: column;
		margin-top: 20rpx;
	}
	
	.item-menu-image {
		width: 230rpx;
		height: 230rpx;
	}
</style>
