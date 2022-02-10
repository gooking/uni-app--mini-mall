<script>
	// #ifdef MP-WEIXIN
	const WXAUTH = require('@/common/wxauth.js')
	// #endif
	export default {
		globalData: {
			subDomain: 'uniminishop',
			version: '1.1.0',
			sysconfigkeys: 'mallName'
		},
		onLaunch() {
			this.$api.init(this.globalData.subDomain)
			const _this = this
			// 1.1.0版本之前关于http拦截器代码，已平滑移动到/common/http.interceptor.js中
			// 注意，需要在/main.js中实例化Vue之后引入如下(详见文档说明)：
			// import httpInterceptor from '@/common/http.interceptor.js'
			// Vue.use(httpInterceptor, app)
			// process.env.VUE_APP_PLATFORM 为通过js判断平台名称的方法，结果分别如下：
			/**
			 * h5，app-plus(nvue下也为app-plus)，mp-weixin，mp-alipay......
			 */
			this.checkForUpdate(); // 检查新版本
			this.queryConfigBatch();
		},
		onShow: function(e) {
			if (e && e.query && e.query.inviter_id) {
				this.$u.vuex('referrer', e.query.inviter_id)
			}
			this.autoLogin()
		},
		onHide: function() {
			// console.log('App Hide，app不再展现在前台')  
		},
		onPageNotFound(e) {
			// 页面不存在 {path: '/1212', query: {a: '123'}, isEntryPage: true}
			console.log(e)
		},
		methods: {
			checkForUpdate() {
				const updateManager = uni.getUpdateManager();
				updateManager.onCheckForUpdate(function(res) {
					// 请求完新版本信息的回调
					// console.log(res.hasUpdate);
				});

				updateManager.onUpdateReady(function(res) {
					uni.showModal({
						title: '更新提示',
						content: '新版本已经准备好，是否重启应用？',
						success(res) {
							if (res.confirm) {
								// 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
								updateManager.applyUpdate();
							}
						}
					});

				});

				updateManager.onUpdateFailed(function(res) {
					// 新的版本下载失败
				});
			},
			async queryConfigBatch() {
				const sysconfigkeys = this.globalData.sysconfigkeys
				if (!sysconfigkeys) {
					return
				}
				const res = await this.$api.queryConfigBatch(sysconfigkeys)
				if (res.code == 0) {
					const sysconfigMap = {}
					res.data.forEach(config => {
						sysconfigMap[config.key] = config.value
					})
					this.$u.vuex('sysconfigMap', sysconfigMap)
				}
			},
			async autoLogin() {
				// 自动登陆
				// #ifdef MP-WEIXIN
				const isLogined = await WXAUTH.checkHasLogined()
				if(!isLogined) {
					await WXAUTH.authorize()
					await WXAUTH.bindSeller()
				}
				setTimeout(() => {
					uni.$emit('loginOK', {})
				}, 500)
				// #endif
			},
			async checkHasLoginedH5() {
				if (!this.token) {
					return false
				}
				// https://www.yuque.com/apifm/nu0f75/mp9f59
				const res = await this.$api.checkToken(this.token)
				if (res.code != 0) {
					this.$u.vuex('token', '')
					return false
				}
				return true
			},
		}
	}
</script>

<style lang="scss">
	@import "uview-ui/index.scss";
</style>
