<template>
	<transition name="slide-right">
		<div class="content">
			<div class="content-wrapper" v-if="bookAvailable">
				<div class="content-item" v-for="(item, index) in navigation.toc" key="index" @click="jumpTo(item.href)">
					<span class="text">{{item.label}}</span>
				</div>
			</div>
			<div class="empty" v-else>加载中...</div>
		</div>
	</transition>
</template>

<script type="text/ecmascript-6">
	export default {
		props: {
			ifShowContent: Boolean,
			navigation: Object,
			bookAvailable: Boolean
		},
		methods: {
			jumpTo(target) {
				this.$emit('jumpTo', target)
			}
		}
	}
</script>
<style lang="scss" scoped>
	@import '../assets/styles/global';
	.content {
		position: absolute;
		top: 0;
		left: 0;
		z-index: 102;
		width: 40%;
		height: 100%;
		padding: px2rem(15);
		display: flex;
		background: white;
		box-shadow: 0 px2rem(8) px2rem(8) rgba(0, 0, 0, .15);
		font-size: px2rem(18);
		overflow: scroll;
		.content-item {
			margin: px2rem(10);
			text-align: left;
			text-decoration: underline;
		}
	}
</style>