<template>
	<div>
		<div class="shopcart">
			<div
				class="content"
				@click="toggleShow"
			>
				<div class="content-left">
					<div class="logo-wrapper">
						<div
							class="logo"
							:class="{highlight:totalNumber>0}"
						>
							<i
								class="iconfont icon-shopping_cart"
								:class="{highlight:totalNumber>0}"
							></i>
						</div>
						<div
							class="num"
							v-show="totalNumber>0"
						>{{totalNumber}}</div>
					</div>
					<div class="price highlight">￥{{totalMoney}}</div>
					<div class="desc">另需配送费￥{{info.deliveryPrice}}元</div>
				</div>
				<div class="content-right">
					<div
						class="pay"
						:class="btnstate >=2 ? 'enough': 'not-enough'"
					>
						{{btnstate===0 ? `￥${info.minPrice}元起送`:(btnstate===1 ? `还差￥${info.minPrice-totalMoney}元起送`:'去结算')}}
					</div>
				</div>
			</div>
			<transition name="move">
				<div
					class="shopcart-list"
					v-show="showCartList && listshow"
				>
					<div class="list-header">
						<h1 class="title">购物车</h1>
						<span
							class="empty"
							@click="clearcart"
						>清空</span>
					</div>
					<div
						class="list-content"
						ref="cartlist"
					>
						<ul>
							<li
								class="food"
								v-for="food in shopcartlist"
								:key="food.name"
							>
								<span class="name">{{food.name}}</span>
								<div class="price"><span>￥{{food.price}}</span></div>
								<div class="cartcontrol-wrapper">
									<CartControl :food="food" />
								</div>
							</li>
						</ul>
					</div>
				</div>
			</transition>
		</div>
		<transition name="fade">
			<div
				class="list-mask"
				v-show="showCartList && listshow"
				@click="toggleShow"
			></div>
		</transition>
	</div>

</template>

<script type="text/ecmascript-6">
	import { mapState, mapGetters } from 'vuex'
	import BScroll from 'better-scroll'
	export default {
		data() {
			return {
				showCartList: false,
			}
		},
		methods: {
			toggleShow() {
				if (this.totalNumber > 0) {
					this.showCartList = !this.showCartList
					this.$nextTick(function () {
						if (!this.listscroll) {
							this.listscroll = new BScroll(this.$refs.cartlist, {
								click: true,
							})
						}
						if (this.totalNumber >= 5) {
							this.listscroll.refresh()
						}
					})
				}
			},
			clearcart() {
				this.$dialog
					.confirm({
						title: '警告',
						message: '是否要全部清空',
						confirmButtonColor: '#4CD96F',
						theme: 'round-button',
					})
					.then(() => {
						this.$store.commit('clear_food_cart')
					})
					.catch(() => {})
			},
		},
		computed: {
			...mapState({
				info: (state) => state.shop.info,
				shopcartlist: (state) => state.shop.shopcartlist,
			}),
			...mapGetters(['totalNumber', 'totalMoney']),
			listshow() {
				if (this.totalNumber <= 0) {
					this.showCartList = false
				}

				return this.totalNumber > 0
			},
			btnstate() {
				if (this.totalNumber === 0) {
					return 0
				} else if (this.totalMoney < this.info.minPrice) {
					return 1
				} else {
					return 2
				}
			},
		},
	}
</script>

<style lang="stylus" rel="stylesheet/stylus" scoped>
	@import '../../common/stylus/mixins.styl';

	.shopcart {
		position: fixed;
		left: 0;
		bottom: 0;
		z-index: 50;
		width: 100%;
		height: 48px;

		.content {
			display: flex;
			background: #141d27;
			font-size: 0;
			color: rgba(255, 255, 255, 0.4);

			.content-left {
				flex: 1;

				.logo-wrapper {
					display: inline-block;
					vertical-align: top;
					position: relative;
					top: -10px;
					margin: 0 12px;
					padding: 6px;
					width: 56px;
					height: 56px;
					box-sizing: border-box;
					border-radius: 50%;
					background: #141d27;

					.logo {
						width: 100%;
						height: 100%;
						border-radius: 50%;
						text-align: center;
						background: #2b343c;

						&.highlight {
							background: $green;
						}

						.icon-shopping_cart {
							line-height: 44px;
							font-size: 24px;
							color: #80858a;

							&.highlight {
								color: #fff;
							}
						}
					}

					.num {
						position: absolute;
						top: 0;
						right: 0;
						width: 24px;
						height: 16px;
						line-height: 16px;
						text-align: center;
						border-radius: 16px;
						font-size: 9px;
						font-weight: 700;
						color: #ffffff;
						background: rgb(240, 20, 20);
						box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.4);
					}
				}

				.price {
					display: inline-block;
					vertical-align: top;
					margin-top: 12px;
					line-height: 24px;
					padding-right: 12px;
					box-sizing: border-box;
					border-right: 1px solid rgba(255, 255, 255, 0.1);
					font-size: 16px;
					font-weight: 700;

					&.highlight {
						color: #fff;
					}
				}

				.desc {
					display: inline-block;
					vertical-align: top;
					margin: 12px 0 0 12px;
					line-height: 24px;
					font-size: 10px;
				}
			}

			.content-right {
				flex: 0 0 105px;
				width: 105px;

				.pay {
					height: 48px;
					line-height: 48px;
					text-align: center;
					font-size: 12px;
					font-weight: 700;

					&.not-enough {
						background: #2b333b;
					}

					&.enough {
						background: #00b43c;
						color: #fff;
					}
				}
			}
		}

		.shopcart-list {
			position: absolute;
			left: 0;
			top: 0;
			z-index: -1;
			width: 100%;
			transform: translateY(-100%);

			&.move-enter-active, &.move-leave-active {
				transition: all 0.5s;
			}

			&.move-enter, &.move-leave-to {
				opacity: 0;
				transform: translateY(0);
			}

			.list-header {
				height: 40px;
				line-height: 40px;
				padding: 0 18px;
				background: #f3f5f7;
				border-bottom: 1px solid rgba(7, 17, 27, 0.1);

				.title {
					float: left;
					font-size: 14px;
					color: rgb(7, 17, 27);
				}

				.empty {
					float: right;
					font-size: 12px;
					color: rgb(0, 160, 220);
				}
			}

			.list-content {
				padding: 0 18px;
				max-height: 217px;
				overflow: hidden;
				background: #fff;

				.food {
					position: relative;
					padding: 12px 0;
					box-sizing: border-box;
					bottom-border-1px(rgba(7, 17, 27, 0.1));

					.name {
						line-height: 24px;
						font-size: 14px;
						color: rgb(7, 17, 27);
					}

					.price {
						position: absolute;
						right: 90px;
						bottom: 12px;
						line-height: 24px;
						font-size: 14px;
						font-weight: 700;
						color: rgb(240, 20, 20);
					}

					.cartcontrol-wrapper {
						position: absolute;
						right: 0;
						bottom: 6px;
					}
				}
			}
		}
	}

	.list-mask {
		position: fixed;
		top: 0;
		left: 0;
		width: 100%;
		height: 100%;
		z-index: 40;
		backdrop-filter: blur(10px);
		opacity: 1;
		background: rgba(7, 17, 27, 0.6);

		&.fade-enter-active, &.fade-leave-active {
			transition: opacity 0.5s;
		}

		&.fade-enter, &.fade-leave-to {
			opacity: 0;
		}
	}
</style>

