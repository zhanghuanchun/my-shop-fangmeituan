<template>
	<section class="loginContainer">
		<div class="loginInner">
			<div class="login_header">
				<h2 class="login_logo">硅谷外卖</h2>
				<div class="login_header_title">
					<a
						href="javascript:;"
						:class="{on:showMesLoginPage}"
						@click="showMesLoginPage=true"
					>短信登录</a>
					<a
						href="javascript:;"
						:class="{on:!showMesLoginPage}"
						@click="showMesLoginPage=false"
					>密码登录</a>
				</div>
			</div>
			<div class="login_content">
				<form>
					<div :class="{on:showMesLoginPage}">
						<section class="login_message">
							<input
								type="tel"
								maxlength="11"
								placeholder="手机号"
								v-model="phone"
								name="phone"
								v-validate="'required|mobile'"
							>
							<button
								:disabled="!isRightPhone || countdown>0"
								class="get_verification"
								:style="{color:isRightPhone && countdown<=0 ?'#000':'#ccc'}"
								@click.prevent="sendCode"
							>{{ countdown>0 ? `短信已发送${countdown}s`:'发送验证码'}}</button>
							<span
								style="color: red;"
								v-show="errors.has('phone')"
							>{{ errors.first('phone') }}</span>
						</section>
						<section class="login_verification">
							<input
								type="text"
								maxlength="8"
								placeholder="验证码"
								v-model="code"
								name="code"
								v-validate="{required: true,regex: /^\d{6}$/}"
							>
							<span
								style="color: red;"
								v-show="errors.has('code')"
							>{{ errors.first('code') }}</span>
						</section>
						<section class="login_hint">
							温馨提示：未注册硅谷外卖帐号的手机号，登录时将自动注册，且代表已同意
							<a href="javascript:;">《用户服务协议》</a>
						</section>
					</div>
					<div :class="{on:!showMesLoginPage}">
						<section>
							<section class="login_message">
								<input
									type="text"
									placeholder="用户名"
									v-model="name"
									name="name"
									v-validate="'required'"
								>
								<span
									style="color: red;"
									v-show="errors.has('name')"
								>{{ errors.first('name') }}</span>
							</section>
							<section class="login_verification">
								<input
									:type="viewpwd ? 'text' : 'password'"
									placeholder="密码"
									v-model="pwd"
									name="pwd"
									v-validate="'required'"
								>
								<div
									class="switch_button"
									:class="{on:viewpwd}"
									@click="viewpwd=!viewpwd"
								>
									<div
										class="switch_circle"
										:style="{transform:viewpwd?'translateX(27px)':'translateX(0px)'}"
									></div>
									<span class="switch_text">{{viewpwd? 'abc':''}}</span>
								</div>
								<span
									style="color: red;"
									v-show="errors.has('pwd')"
								>{{ errors.first('pwd') }}</span>
							</section>
							<section class="login_message">

								<input
									type="text"
									maxlength="11"
									placeholder="验证码"
									v-model="captcha"
									name="captcha"
									v-validate="{required: true,regex: /^[0-9a-zA-Z]{4}$/}"
								>

								<img
									class="get_verification"
									src="http://localhost:4000/captcha"
									alt="captcha"
									@click="reqCaptcha"
									ref="captcha_img"
								>
								<span
									style="color: red;"
									v-show="errors.has('captcha')"
								>{{ errors.first('captcha') }}</span>
							</section>
						</section>
					</div>
					<button
						class="login_submit"
						@click.prevent="login"
					>{{$t('login_login')}}</button>
				</form>
				<a
					href="javascript:;"
					class="about_us"
				>{{$t('login_aboutUs')}}</a>
			</div>
			<a
				href="javascript:"
				class="go_back"
			>
				<i class="iconfont icon-jiantou2"></i>
			</a>
			<van-button
				@click="switchlanguage"
				size="mini"
				plain
				type="primary"
			>切换语言</van-button>
		</div>
	</section>

</template>

<script type="text/ecmascript-6">
	export default {
		data() {
			return {
				showMesLoginPage: false,
				viewpwd: false,
				phone: '',
				code: '',
				name: '', // 用户名
				pwd: '', // 密码
				captcha: '', // 图形验证码
				countdown: 0,
			}
		},
		computed: {
			isRightPhone() {
				return /^1[3579]\d{9}$/.test(this.phone)
			},
		},
		methods: {
			async sendCode() {
				this.countdown = 10
				this.countId = setInterval(() => {
					if (this.countdown <= 0) {
						clearInterval(this.countId)
					}
					this.countdown--
				}, 1000)
				const result = await this.$API.reqPhoneCode(this.phone)
				const { code, msg } = result
				if (code === 0) this.$toast.success('发送成功')
				else {
					clearInterval(this.countId)
					this.countdown = 0
					this.$toast.fail(msg)
				}
			},
			async login() {
				// this.reqCaptcha()
				let names
				if (this.showMesLoginPage) names = ['phone', 'code']
				else names = ['name', 'pwd', 'captcha']

				let validateresult = await this.$validator.validateAll(names)
				if (validateresult) {
					const { phone, code, name, pwd, captcha } = this
					let result
					if (this.showMesLoginPage)
						result = await this.$API.reqSmslogin(phone, code)
					else result = await this.$API.reqPwdlogin(name, pwd, captcha)
					const { data, msg } = result
					const status = result.code
					if (status === 0) {
						this.$store.dispatch('saveUserInfo', data)
						this.$router.replace({ path: '/profile' })
					} else {
						this.$dialog.alert({
							title: '提示',
							message: msg,
							confirmButtonColor: '#4CD96F',
							theme: 'round-button',
						})
					}
				} else {
					this.$dialog.alert({
						title: '提示',
						message: '输入内容格式不正确，请检查！',
						confirmButtonColor: '#4CD96F',
						theme: 'round-button',
					})
				}
				this.reqCaptcha()
			},
			reqCaptcha() {
				this.$refs.captcha_img.src =
					'http://localhost:4000/captcha?' + Date.now()
			},
			switchlanguage() {
				// 根据当前语言得到新的语言
				const locale = this.$i18n.locale === 'en' ? 'zh_CN' : 'en'
				// 指定新的语言
				this.$i18n.locale = locale
				// 将新的语言保存到local
				localStorage.setItem('locale_key', locale)
			},
		},
		beforeRouteEnter(to, from, next) {
			// 在渲染该组件的对应路由被 confirm 前调用
			// 不！能！获取组件实例 `this`
			// 因为当守卫执行前，组件实例还没被创建
			next((vm) => {
				// 通过 `vm` 访问组件实例
				if (vm.$store.state.user.token) next('/profile')
				else next()
			})
		},
	}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
	@import '../../common/stylus/mixins.styl';

	.loginContainer {
		width: 100%;
		height: 100%;
		background: #fff;

		.loginInner {
			padding-top: 60px;
			width: 80%;
			margin: 0 auto;

			.login_header {
				.login_logo {
					font-size: 40px;
					font-weight: bold;
					color: #02a774;
					text-align: center;
				}

				.login_header_title {
					padding-top: 40px;
					text-align: center;

					>a {
						color: #333;
						font-size: 14px;
						padding-bottom: 4px;

						&:first-child {
							margin-right: 40px;
						}

						&.on {
							color: #02a774;
							font-weight: 700;
							border-bottom: 2px solid #02a774;
						}
					}
				}
			}

			.login_content {
				>form {
					>div {
						display: none;

						&.on {
							display: block;
						}

						input {
							width: 100%;
							height: 100%;
							padding-left: 10px;
							box-sizing: border-box;
							border: 1px solid #ddd;
							border-radius: 4px;
							outline: 0;
							font: 400 14px Arial;

							&:focus {
								border: 1px solid #02a774;
							}
						}

						.login_message {
							position: relative;
							margin-top: 16px;
							height: 48px;
							font-size: 14px;
							background: #fff;

							.get_verification {
								position: absolute;
								top: 50%;
								right: 10px;
								transform: translateY(-50%);
								border: 0;
								color: #ccc;
								font-size: 14px;
								background: transparent;
							}
						}

						.login_verification {
							position: relative;
							margin-top: 16px;
							height: 48px;
							font-size: 14px;
							background: #fff;

							.switch_button {
								font-size: 12px;
								border: 1px solid #ddd;
								border-radius: 8px;
								transition: background-color 0.3s, border-color 0.3s;
								padding: 0 6px;
								width: 30px;
								height: 16px;
								line-height: 16px;
								color: #fff;
								position: absolute;
								top: 50%;
								right: 10px;
								transform: translateY(-50%);

								&.off {
									background: #fff;

									.switch_text {
										float: right;
										color: #ddd;
									}
								}

								&.on {
									background: #02a774;
								}

								>.switch_circle {
									// transform translateX(27px)
									position: absolute;
									top: -1px;
									left: -1px;
									width: 16px;
									height: 16px;
									border: 1px solid #ddd;
									border-radius: 50%;
									background: #fff;
									box-shadow: 0 2px 4px 0 rgba(0, 0, 0, 0.1);
									transition: transform 0.3s;
								}
							}
						}

						.login_hint {
							margin-top: 12px;
							color: #999;
							font-size: 14px;
							line-height: 20px;

							>a {
								color: #02a774;
							}
						}
					}

					.login_submit {
						display: block;
						width: 100%;
						height: 42px;
						margin-top: 30px;
						border-radius: 4px;
						background: #4cd96f;
						color: #fff;
						text-align: center;
						font-size: 16px;
						line-height: 42px;
						border: 0;
					}
				}

				.about_us {
					display: block;
					font-size: 12px;
					margin-top: 20px;
					text-align: center;
					color: #999;
				}
			}

			.go_back {
				position: absolute;
				top: 5px;
				left: 5px;
				width: 30px;
				height: 30px;

				>.iconfont {
					font-size: 20px;
					color: #999;
				}
			}
		}
	}
</style>
