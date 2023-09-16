<template>
	<view v-if="projectsApiOK && (!projects || projects.length == 0)">
		<page-box-empty title="暂未添加公司" sub-title="请联系管理员为您添加公司信息" :show-btn="false" />
	</view>
	<view v-else-if="projectsApiOK">
		<picker @change="projectChange" :value="projectIndex" :range="projects" range-key="name">
			<uni-list>
				<uni-list-item :title="project.name" rightText="切换公司" clickable showArrow></uni-list-item>
			</uni-list>
		</picker>
		<view class="pd32">
			<uni-segmented-control :current="current" :values="items" @clickItem="onClickItem" styleType="button"></uni-segmented-control>
		</view>
		<uni-list v-if="current == 0">
			<uni-list-item title="统计时间" :rightText="dayBegin" clickable showArrow @click="changeDay"></uni-list-item>
		</uni-list>
		<picker v-else-if="current == 1" @change="weekChange" :value="weekIndex" :range="weekList">
			<uni-list>
				<uni-list-item title="统计时间" :rightText="'第' + (weekIndex + 1) + '周' + ' ' + dayBegin + ' ~ ' + dayEnd" clickable showArrow></uni-list-item>
			</uni-list>
		</picker>
		<picker v-else-if="current == 2" @change="monthChange" :value="monthIndex" :range="monthList">
			<uni-list>
				<uni-list-item title="统计时间" :rightText="(monthIndex + 1) + '月' + ' ' + dayBegin + ' ~ ' + dayEnd" clickable showArrow></uni-list-item>
			</uni-list>
		</picker>
		<page-box-empty v-if="!list || list.length == 0" title="暂无工时统计" :show-btn="false" />
		<uni-list>
			<uni-list-item v-for="(item,index) in list" :key="item.uid" :title="item.nick" :rightText="item.number + '小时'"></uni-list-item>
		</uni-list>
		<uni-calendar
			ref="calendar"
			:insert="false"
			:lunar="true"
			:date="dayBegin"
			:end-date="today"
			@confirm="calendarconfirm"
		 />
	</view>
</template>

<script>
	const WXAUTH = require('@/common/wxauth.js')
	const dayjs = require('dayjs')
	import "dayjs/locale/zh-cn";
	dayjs.locale('zh-cn')
	const weekOfYear = require('dayjs/plugin/weekOfYear')
	dayjs.extend(weekOfYear)
	const isoWeeksInYear = require('dayjs/plugin/isoWeeksInYear')
	const isLeapYear = require('dayjs/plugin/isLeapYear') // dependent on isLeapYear plugin
	dayjs.extend(isoWeeksInYear)
	dayjs.extend(isLeapYear)
	export default {
		data() {
			return {
				items: ['按天统计', '按周统计', '按月统计'],
				current: 0,
				id: undefined,
				projects: undefined,
				projectsApiOK: false,
				projectIndex: 0,
				project: undefined,
				list: undefined,
				today: dayjs().format('YYYY-MM-DD'),
				dayBegin: dayjs().format('YYYY-MM-DD'),
				dayEnd: dayjs().format('YYYY-MM-DD'),
				totalWeeks: dayjs().isoWeeksInYear(), // 一年总共有几周
				weekIndex: dayjs().week() - 1, // 第几周
				weekList: [],
				monthIndex: dayjs().month(),
				monthList:[],
			}
		},
		onLoad(e) {
			// this.id = e.id
			// // #ifdef MP-WEIXIN
			// if (e && e.scene) {
			//   const scene = decodeURIComponent(e.scene)
			//   if (scene) {
			// 	this.id = scene
			//   }
			// }
			// // #endif
			for (let i = 1; i <= dayjs().week(); i++) {
				this.weekList.push(`第${i}周`)
			}
			for (let i = 0; i <= dayjs().month(); i++) {
				this.monthList.push(`${i + 1}月`)
			}
			this.workingHoursBossReport()
		},
		onShow() {
			
		},
		methods: {
			onClickItem(e) {
			  this.current = e.currentIndex
			  if (this.current == 0) {
			  	// 按天
			  	this.dayBegin = dayjs().format('YYYY-MM-DD')
			  	this.dayEnd = dayjs().format('YYYY-MM-DD')
			  }
			  if (this.current == 1) {
			  	// 按周
				this.weekIndex = dayjs().week() - 1
			  	this.dayBegin = dayjs().week(this.weekIndex + 1).startOf('week').format('YYYY-MM-DD')
			  	this.dayEnd = dayjs().week(this.weekIndex + 1).endOf('week').format('YYYY-MM-DD')
			  }
			  if (this.current == 2) {
			  	// 按月
				this.monthIndex = dayjs().month()
			  	this.dayBegin = dayjs().month(this.monthIndex).startOf('month').format('YYYY-MM-DD')
			  	this.dayEnd = dayjs().month(this.monthIndex).endOf('month').format('YYYY-MM-DD')
			  }
			  this.workingHoursBossReport()
			},
			projectChange(e) {
				this.projectIndex = e.detail.value
				this.id = this.projects[e.detail.value].id
				this.project = this.projects[e.detail.value]
				this.workingHoursBossReport()
			},
			async workingHoursBossReport() {
				const isLogined = await WXAUTH.checkHasLogined()
				if (!isLogined) {
					setTimeout(() => {
						this.workingHoursBossReport()
					}, 500)
					return
				}
				const res2 = await this.$api.workingHoursMyProject({
					token: this.token
				})
				this.projectsApiOK = true
				if (res2.code == 0) {
					this.projects = res2.data.result
					if (!this.id) {
						this.projectIndex = 0
						this.id = this.projects[0].id
						this.project = this.projects[0]
					}
				}
				uni.showLoading({
					title: ''
				})
				const d = {
					token: this.token,
					projectId: this.id || '',
					dayBegin: this.dayBegin,
					dayEnd: this.dayEnd,
				}
				const res = await this.$api.workingHoursBossReport(d)
				uni.hideLoading()
				if (res.code != 0) {
					this.list = null
					return
				}
				// this.project = res.data.project
				res.data.result.forEach(ele => {
					const userInfo = res.data.userMap[ele.uid]
					if (userInfo) {
						ele.nick = userInfo.nick
					}
				})
				this.list = res.data.result
			},
			changeDay() {
				if (this.current == 0) {
					// 按天
					this.$refs.calendar.open()
				}
				if (this.current == 2) {
					// 按月
					d.dayBegin = dayjs().startOf('month').format('YYYY-MM-DD')
				}
			},
			calendarconfirm(e) {
				this.dayBegin = e.fulldate
				this.dayEnd = e.fulldate
				this.workingHoursBossReport()
			},
			weekChange(e) {
				this.weekIndex = e.detail.value * 1
				this.dayBegin = dayjs().week(this.weekIndex + 1).startOf('week').format('YYYY-MM-DD')
				this.dayEnd = dayjs().week(this.weekIndex + 1).endOf('week').format('YYYY-MM-DD')
				this.workingHoursBossReport()
			},
			monthChange(e) {
				this.monthIndex = e.detail.value * 1
				this.dayBegin = dayjs().month(this.monthIndex).startOf('month').format('YYYY-MM-DD')
				this.dayEnd = dayjs().month(this.monthIndex).endOf('month').format('YYYY-MM-DD')
				this.workingHoursBossReport()
			},
		}
	}
</script>

<style lang="scss" scoped>
	.title {
		text-align: center;
		font-weight: bold;
		padding: 32rpx;
	}
</style>
