<template>
	<div
		class="star"
		:class="size"
	>
		<span
			class="star-item"
			v-for="(rating,index) in ratingArr"
			:key="index"
			:class="rating"
		></span>
	</div>
</template>

<script type="text/ecmascript-6">
	export default {
		props: {
			size: String,
			rating: Number,
		},
		computed: {
			ratingArr() {
				let ratingArr = []
				let on = (this.rating * 10) / 10
				let half = (this.rating * 10) % 10
				for (let index = 1; index <=on; index++) {
					ratingArr.push('on')
				}
				if (half > 5) ratingArr.push('half')
				while (ratingArr.length < 5) ratingArr.push('off')
				return ratingArr
			},
		},
	}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
	@import '../../common/stylus/mixins.styl';

	.star { // 2x图 3x图
		float: left;
		font-size: 0;

		.star-item {
			display: inline-block;
			background-repeat: no-repeat;
		}

		&.star-48 {
			.star-item {
				width: 20px;
				height: 20px;
				margin-right: 22px;
				background-size: 20px 20px;

				&:last-child {
					margin-right: 0;
				}

				&.on {
					bg-image('./images/stars/star48_on');
				}

				&.half {
					bg-image('./images/stars/star48_half');
				}

				&.off {
					bg-image('./images/stars/star48_off');
				}
			}
		}

		&.star-36 {
			.star-item {
				width: 15px;
				height: 15px;
				margin-right: 6px;
				background-size: 15px 15px;

				&:last-child {
					margin-right: 0;
				}

				&.on {
					bg-image('./images/stars/star36_on');
				}

				&.half {
					bg-image('./images/stars/star36_half');
				}

				&.off {
					bg-image('./images/stars/star36_off');
				}
			}
		}

		&.star-24 {
			.star-item {
				width: 10px;
				height: 10px;
				margin-right: 3px;
				background-size: 10px 10px;

				&:last-child {
					margin-right: 0;
				}

				&.on {
					bg-image('./images/stars/star24_on');
				}

				&.half {
					bg-image('./images/stars/star24_half');
				}

				&.off {
					bg-image('./images/stars/star24_off');
				}
			}
		}
	}
</style>
