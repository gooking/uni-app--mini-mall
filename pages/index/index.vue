<template>
	<view class="Index">
		<view class="blank-top"></view>
		<view class="title">
			{{ sysconfigMap.mallName }}
			<text></text>
		</view>
		<view class="title-sub">推荐商品</view>
		<u-swiper class="swiper" :list="goodsList" name="pic" :title="true" :effect3d="true" height="600"
			@change="goodschange" @click="swipeclick"></u-swiper>
		<view v-if="curGoods" class="cur-goods">
			<view class="amount"><text>Ұ</text>{{ curGoods.minPrice }}</view>
			<view class="btns">
				<u-icon class="btn" name="shopping-cart" @click="addCart"></u-icon>
				<u-icon class="btn" :name="curGoods.curUserHasFaved ? 'heart-fill' : 'heart'" @click="fav"></u-icon>
			</view>
		</view>

		<u-popup v-model="goodsShow" mode="bottom" border-radius="14" safe-area-inset-bottom closeable>
			<view v-if="selectGoods" class="goodsShow">
				<view class="pic-amount">
					<u-image width="300rpx" height="300rpx" mode="widthFix" :src="selectGoodsPic"></u-image>
					<view class="amount"><text>Ұ</text>{{ selectGoodsPrice }}</view>
				</view>
				<div v-for="(item, index) in selectGoods.properties" :key="item.id">
					<u-section :title="item.name" :right="false"></u-section>
					<view class="tags">
						<u-tag v-for="(item2,index2) in item.childsCurGoods" :key="item2.id"
							:class="item2.hidden ? 'item hidden' : 'item'" :text="item2.name" mode="plain"
							:type="item2.active ? 'success' : 'info'" @click="skuclick(index, index2)" />
					</view>
				</div>
				<div v-for="(item, index) in selectGoodsAddition" :key="item.id">
					<u-section :title="item.name" :right="false"></u-section>
					<view class="tags">
						<u-tag v-for="(item2,index2) in item.items" :key="item2.id" class="item" :text="item2.name"
							mode="plain" :type="item2.active ? 'success' : 'info'"
							@click="additionclick(index, index2)" />
					</view>
				</div>
				<u-section title="购买数量">
					<u-number-box slot="right" class="bjq" v-model="buyNumber" :min="buyNumMin" :max="buyNumMax">
					</u-number-box>
				</u-section>
				<u-button class="submit" type="success" :disabled="buyNumber == 0 ? true : false" @click="addCart2">
					加入购物车</u-button>
			</view>
		</u-popup>
	</view>
</template>



<script>
	export default {
		data() {
			return {
				goodsList: undefined,
				curGoods: undefined,
				goodsShow: false,
				selectGoods: undefined,
				selectGoodsPic: undefined,
				selectGoodsPrice: undefined,
				selectGoodsCanSubmit: false,
				selectGoodsPropertyChildIds: undefined,
				selectGoodsPropertyChildNames: undefined,
				selectGoodsAddition: undefined,
				buyNumber: 1,
				buyNumMin: 0,
				buyNumMax: 1,
			}
		},
		onLoad() {
			// #ifdef MP-WEIXIN
			uni.$once('loginOK', data => {
				this.remommendGoods()
			})
			// #endif
			// #ifndef MP-WEIXIN
			this.remommendGoods()
			// #endif
		},
		onShow() {

		},
		created() {},
		methods: {
			async remommendGoods(append) {
				const data = {
					token: this.token,
					recommendStatus: 1
				}
				if(await getApp().checkHasLoginedH5()) {
					data.hasFavedFlag = true
				}
				const res = await this.$api.goodsv2(data)
				if (res.code != 0) {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
				} else {
					res.data.result.forEach(ele => {
						ele.title = ele.name
					})
					this.goodsList = res.data.result
					this.curGoods = res.data.result[0]
				}
			},
			goodschange(index) {
				this.curGoods = this.goodsList[index]
			},
			async addCart() {
				if(!await getApp().checkHasLoginedH5()) {
					uni.navigateTo({
						url: '/pages/login/login'
					})
					return
				}
				if (this.curGoods.propertyIds || this.curGoods.hasAddition) {
					// 需要选择sku
					const res = await this.$api.goodsDetail(this.curGoods.id, this.token)
					if (res.code != 0) {
						uni.showToast({
							title: res.msg,
							icon: 'none'
						})
						return
					}
					this.selectGoods = res.data
					this.selectGoodsPic = res.data.basicInfo.pic
					this.selectGoodsPrice = res.data.basicInfo.minPrice
					if (res.data.basicInfo.hasAddition) {
						// 获取可选配件
						this.goodsAddition()
					}
					this.goodsShow = true
				} else {
					// 直接加入购物车
					const res = await this.$api.shippingCarInfoAddItem(this.token, this.curGoods.id, this.curGoods.minBuyNumber)
					if (res.code != 0) {
						uni.showToast({
							title: res.msg,
							icon: 'none'
						})
						return
					}
					uni.showToast({
						title: '已加入购物车'
					})
				}
			},
			async addCart2() {
				const goodsAddition = []
				if (this.selectGoods.basicInfo.hasAddition) {
					let canSubmit = true
					this.selectGoodsAddition.forEach(ele => {
						if (ele.required) {
							const a = ele.items.find(item => {
								return item.active
							})
							if (!a) {
								canSubmit = false
							}
						}
						ele.items.forEach(item => {
							if (item.active) {
								goodsAddition.push({
									id: item.id,
									pid: item.pid
								})
							}
						})
					})
					if (!canSubmit) {
						uni.showToast({
							title: '请选择配件',
							icon: 'none'
						})
						return
					}
				}
				if (this.buyNumber < 1) {
					uni.showToast({
						title: '请选择购买数量',
						icon: 'none'
					})
					return
				}
				const sku = []
				if (this.selectGoods.properties) {
					this.selectGoods.properties.forEach(p => {
						sku.push({
							optionId: p.id,
							optionValueId: p.optionValueId
						})
					})
				}
				const res = await this.$api.shippingCarInfoAddItem(this.token, this.selectGoods.basicInfo.id, this.buyNumber, sku, goodsAddition)
				if (res.code != 0) {
					uni.showToast({
						title: res.msg,
						icon: 'none'
					})
					return
				}
				this.goodsShow = false
				uni.showToast({
					title: '加入购物车',
					icon: 'success'
				})
			},
			async skuclick(index1, index2) {
				const property = this.selectGoods.properties[index1]
				const child = property.childsCurGoods[index2]
				// 当前位置往下的所有sku取消选中状态
				for (let index = index1; index < this.selectGoods.properties.length; index++) {
					const element = this.selectGoods.properties[index]
					element.optionValueId = null
					element.childsCurGoods.forEach(child => {
						child.active = false
					})
				}
				// 设置当前选中状态，或者取消选中
				property.optionValueId = child.id
				child.active = true
				// 获取所有的选中规格尺寸数据
				const needSelectNum = this.selectGoods.properties.length
				let curSelectNum = 0;
				let propertyChildIds = "";
				let propertyChildNames = "";
				let _skuList = this.selectGoods.skuList

				this.selectGoods.properties.forEach(p => {
					p.childsCurGoods.forEach(c => {
						// 处理当前选中的sku信息
						if (c.active) {
							_skuList = _skuList.filter(aaa => {
								return aaa.propertyChildIds.indexOf(p.id + ':' + c.id) != -1
							})
							curSelectNum++;
							propertyChildIds = propertyChildIds + p.id + ":" + c.id + ",";
							propertyChildNames = propertyChildNames + p.name + ":" + c.name + "  ";
						} else if (!p.optionValueId) {
							const nextO = _skuList.find(aaa => {
								return aaa.propertyChildIds.indexOf(p.id + ':' + c.id) != -1
							})
							c.hidden = nextO ? false : true
						}
					})
				})
				this.selectGoodsCanSubmit = false;
				if (needSelectNum == curSelectNum) {
					this.selectGoodsCanSubmit = true;
				}
				if (this.selectGoods.subPics && this.selectGoods.subPics.length > 0) {
					const _subPic = this.selectGoods.subPics.find(ele => {
						return ele.optionValueId == child.id
					})
					if (_subPic) {
						this.selectGoodsPic = _subPic.pic
					}
				}
				this.selectGoodsPropertyChildIds = propertyChildIds
				this.selectGoodsPropertyChildNames = propertyChildNames
				this.$forceUpdate()
				this.calculateGoodsPrice()
			},
			async additionclick(index1, index2) {
				const property = this.selectGoodsAddition[index1]
				const child = property.items[index2]
				if (child.active) {
					// 该操作为取消选择
					child.active = false
					this.selectGoodsAddition.splice(index1, 1, property)
					this.calculateGoodsPrice()
					return
				}
				// 单选配件取消所有子栏目选中状态
				if (property.type == 0) {
					property.items.forEach(child => {
						child.active = false
					})
				}
				// 设置当前选中状态
				child.active = true
				this.selectGoodsAddition.splice(index1, 1, property)
				this.calculateGoodsPrice()
			},
			async calculateGoodsPrice() {
				// 计算最终的商品价格
				let price = this.selectGoods.basicInfo.minPrice
				let originalPrice = this.selectGoods.basicInfo.originalPrice
				let totalScoreToPay = this.selectGoods.basicInfo.minScore
				let buyNumMax = this.selectGoods.basicInfo.stores
				let buyNumber = this.selectGoods.basicInfo.minBuyNumber
				// if (this.data.shopType == 'toPingtuan') {
				// 	price = this.selectGoods.basicInfo.pingtuanPrice
				// }
				// 计算 sku 价格
				if (this.selectGoodsCanSubmit) {
					const res = await this.$api.goodsPriceV2({
						token: this.token,
						goodsId: this.selectGoods.basicInfo.id,
						propertyChildIds: this.selectGoodsPropertyChildIds
					})
					if (res.code == 0) {
						price = res.data.price
						// if (this.data.shopType == 'toPingtuan') {
						// 	price = res.data.pingtuanPrice
						// }
						originalPrice = res.data.originalPrice
						totalScoreToPay = res.data.score
						buyNumMax = res.data.stores
					}
				}
				// 计算配件价格
				if (this.selectGoodsAddition) {
					this.selectGoodsAddition.forEach(big => {
						big.items.forEach(small => {
							if (small.active) {
								price = (price * 100 + small.price * 100) / 100
							}
						})
					})
				}
				this.selectGoodsPrice = price // originalPrice totalScoreToPay
				this.buyNumMax = buyNumMax
				this.buyNumber = (buyNumMax > buyNumber) ? buyNumber : 0
			},
			async goodsAddition() {
				const res = await this.$api.goodsAddition(this.selectGoods.basicInfo.id)
				if (res.code == 0) {
					this.selectGoodsAddition = res.data
				}
			},
			async fav() {
				if(!await getApp().checkHasLoginedH5()) {
					uni.navigateTo({
						url: '/pages/login/login'
					})
					return
				}
				if (this.curGoods.curUserHasFaved) {
					// 取消收藏
					const res = await this.$api.goodsFavDelete(this.token, '', this.curGoods.id)
					if (res.code == 0) {
						uni.showToast({
							title: '取消成功'
						})
						this.curGoods.curUserHasFaved = false
					}
				} else {
					// 加入收藏
					const res = await this.$api.goodsFavPut(this.token, this.curGoods.id)
					if (res.code == 0) {
						uni.showToast({
							title: '收藏成功'
						})
						this.curGoods.curUserHasFaved = true
					}
				}
			},
			swipeclick(index) {
				// console.log(index);
				uni.navigateTo({
					url: '../goods/details?id=' + this.curGoods.id
				})
			},
		}
	}
</script>
<style scoped lang="scss">
	.blank-top {
		height: 88rpx;
	}
	.title {
		font-size: 90rpx;
		margin-left: 20px;
		color: #293539;
		font-weight: 300;
		position: relative;

		text {
			width: 7px;
			height: 7px;
			position: absolute;
			border: 2px solid #a78845;
			border-radius: 50%;
		}
	}

	.title-sub {
		margin-left: 25px;
		color: #293539;
		font-size: 36rpx;
		margin-top: 24px;
		font-weight: 300;
	}

	.cur-goods {
		margin-top: 48rpx;

		.amount {
			text-align: center;
			font-size: 68rpx;
			color: #a78845;

			text {
				font-size: 48rpx;
				padding-right: 3px;
			}
		}

		.btns {
			margin-top: 64rpx;
			display: flex;
			justify-content: space-around;

			.btn {
				color: #a78845;
				font-size: 64rpx;
				background-color: #e3e9fc;
				border-radius: 50%;
				padding: 24rpx;
			}
		}
	}

	.goodsShow {
		.pic-amount {
			display: flex;
			align-items: center;

			.amount {
				margin-left: 64rpx;
				font-size: 68rpx;
				color: #a78845;

				text {
					font-size: 48rpx;
					padding-right: 3px;
				}
			}
		}

		padding: 32rpx;

		.tags {
			margin-top: 32rpx;
			padding-bottom: 48rpx;

			.item {
				margin: 8rpx 4rpx;
			}

			.hidden {
				display: none;
			}
		}

		.submit {
			margin-top: 32rpx;
		}
	}
</style>
