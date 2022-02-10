<template>
	<view class="goods-fav">
		<page-box-empty v-if="!list || list.length == 0" title="暂无收藏的商品" sub-title="可以去看看有那些想买的～" :show-btn="true" />
		<view class="wrap">
			<u-waterfall v-model="list" ref="uWaterfall">
				<template v-slot:left="{leftList}">
					<view class="demo-warter" v-for="(item, index) in leftList" :key="index">
						<!-- 警告：微信小程序中需要hx2.8.11版本才支持在template中结合其他组件，比如下方的lazy-load组件 -->
						<u-lazy-load threshold="-450" border-radius="10" :image="item.pic" :index="index" @click="goGoods(item.goodsId)"></u-lazy-load>
						<view class="demo-title" @click="goGoods(item.goodsId)">
							{{item.goodsName}}
						</view>
						<u-icon name="close-circle-fill" color="#fa3534" size="34" class="u-close" @click="remove(item.id)"></u-icon>
					</view>
				</template>
				<template v-slot:right="{rightList}">
					<view class="demo-warter" v-for="(item, index) in rightList" :key="index">
						<u-lazy-load threshold="-450" border-radius="10" :image="item.pic" :index="index" @click="goGoods(item.goodsId)"></u-lazy-load>
						<view class="demo-title" @click="goGoods(item.goodsId)">
							{{item.goodsName}}
						</view>
						<u-icon name="close-circle-fill" color="#fa3534" size="34" class="u-close" @click="remove(item.id)"></u-icon>
					</view>
				</template>
			</u-waterfall>
		</view>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				list: [],
			}
		},
		created() {
		
		},
		mounted() {
			
		},
		onReady() {
			
		},
		onLoad(e) {
			this._goodsFavList()
		},
		onShow() {

		},
		methods: {
			async _goodsFavList() {
				const res = await this.$api.goodsFavList({
					token: this.token
				})
				if(res.code == 0) {
					// this.$refs.uWaterfall.clear();
					this.list = res.data
					// res.data.forEach(ele => {
					// 	this.list.push(ele)
					// })
				}
			},
			goGoods(id) {
				uni.navigateTo({
					url: './details?id=' + id
				})
			},
			async remove(id) {
				uni.showModal({
				    title: '请确认',
				    content: '确定要取消收藏该商品吗？',
				    success: res => {
				        if (res.confirm) {
				            this._remove(id)
				        }
				    }
				});
			},
			async _remove(id) {
				const res = await this.$api.goodsFavDeleteV2({
					token: this.token,
					id
				})
				if(res.code != 0) {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
				} else {
					uni.showToast({
						title: '删除成功'
					})
					this.$refs.uWaterfall.remove(id)
				}
			}
		}
	}
</script>
<style scoped lang="scss">
.goods-fav {
	.empty {
		padding-top: 88rpx;
	}
	.demo-warter {
		border-radius: 8px;
		margin: 5px;
		background-color: #ffffff;
		padding: 8px;
		position: relative;
		.u-close {
			position: absolute;
			top: 32rpx;
			right: 32rpx;
		}
		.demo-title {
			text-align: center;
			font-size: 26rpx;
			margin-top: 5px;
			color: $u-main-color;
		}
	}
}
</style>
