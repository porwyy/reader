<template>
	<div class='ebook'>
		<title-bar :ifTitleAndMenuShow="ifTitleAndMenuShow"></title-bar>
		<div class='read-wrapper'>
			<div id='read'></div>
			<div class='mask'>
				<div class="left" @click="prevPage"></div>
				<div class="center" @click="toggleTitleAndMenu"></div>
				<div class="right" @click="nextPage"></div>
			</div>
		</div>
		<menu-bar   :ifTitleAndMenuShow="ifTitleAndMenuShow" 
					:fontSizeList="fontSizeList" 
					:defaultFontSize="defaultFontSize"
					@setFontSize="setFontSize"
					:themeList="themeList"
					:defaultTheme="defaultTheme"
					@setTheme="setTheme"
					:bookAvailable="bookAvailable"
					@onProgressChange="onProgressChange"
					:navigation="navigation"
					@jumpTo="jumpTo"
					ref="menuBar"></menu-bar>
	</div>
</template>
<script>
	import Epub from 'epubjs'
	import TitleBar from '@/components/TitleBar'
	import MenuBar from '@/components/MenuBar'
	const DOWNLAD_URL = '/static/一个人的朝圣.epub'
	global.epub = Epub
	export default {
		components: {
			TitleBar,
			MenuBar
		},
		data() {
			return {
				ifTitleAndMenuShow: false,
				fontSizeList: [
					{ fontSize: 12 },
					{ fontSize: 14 },
					{ fontSize: 16 },
					{ fontSize: 18 },
					{ fontSize: 20 },
					{ fontSize: 22 },
					{ fontSize: 24 },
				],
				defaultFontSize: 16,
				themeList: [
					{
						name: 'default',
						style: {
							body: {
								'color': '#000', 
								'background': '#fff'
							}
						}
					},
					{
						name: 'eye',
						style: {
							body: {
								'color': '#000', 
								'background': '#ceeaba'
							}
						}
					},
					{
						name: 'night',
						style: {
							body: {
								'color': '#fff', 
								'background': '#000'
							}
						}
					},
					{
						name: 'gold',
						style: {
							body: {
								'color': '#000', 
								'background': 'rgb(241,236,226)'
							}
						}
					}
				],
				defaultTheme: 0,
				//图书是否处于可用状态
				bookAvailable: false,
				navigation: {}
			}
		},
		methods: {
			jumpTo(href) {
				this.rendition.display(href)
				this.hideTitleAndMenu()
			},
			hideTitleAndMenu() {
				this.ifTitleAndMenuShow = false
				this.$refs.menuBar.hideSetting()
				this.$refs.menuBar.hideContent()
			},
			//精度条progress为0-100
			onProgressChange(progress) {
				const percentage = progress / 100;
				const location = percentage > 0 ? this.locations.cfiFromPercentage(percentage): 0
				this.rendition.display(location)
			},
			setTheme(index) {
				this.themes.select(this.themeList[index].name)
				//选择样式
				this.defaultTheme = index
			},
			registerTheme(){
				this.themeList.forEach(theme => {
					//注册样式
					this.themes.register(theme.name, theme.style)
				})
			},
			setFontSize(fontSize) {
				this.defaultFontSize = fontSize
				if(this.themes) {
					this.themes.fontSize(fontSize + 'px')
				}
			},
			toggleTitleAndMenu() {
				this.ifTitleAndMenuShow = !this.ifTitleAndMenuShow
				if(!this.ifTitleAndMenuShow) {
					this.$refs.menuBar.hideSetting()
				}
			},
			prevPage() {
				//调用rendition的prev方法返回上一页
				if(this.rendition) {
					this.rendition.prev();
				}
			},
			nextPage(){
				//调用rendition的next方法返回下一页
				if(this.rendition) {
					this.rendition.next();
				}
			},
			//渲染电子书
			showEpub() {
				//生成book
				this.book = new Epub(DOWNLAD_URL)
				//生成Rendtion,book.renderTo构成
				this.rendition = this.book.renderTo('read',{
					width: window.innerWidth,
					height: window.innerHeight
				})
				//通过Rendtion.display渲染电子书
				this.rendition.display()
				//获取Theme对象
				this.themes = this.rendition.themes
				//设置默认字体
				this.setFontSize(this.defaultFontSize)
				//this.themes.register(name,styles)
				//this.themes.select(name)
				this.registerTheme()
				this.setTheme(this.defaultTheme)
				//用epubjs的自幻术构建location对象
				this.book.ready.then(() => {
					this.navigation = this.book.navigation
					console.log(this.navigation)
					return this.book.locations.generate()
				}).then(result => {
					this.locations = this.book.locations
					this.bookAvailable = true
				})
			}
		},
		mounted() {
			this.showEpub()
		}
	}
</script>
<style lang='scss' scoped>
	@import 'assets/styles/global';
	.ebook {
		position: relative;
		.read-wrapper {
			.mask {
				position: absolute;
				top: 0;
				left: 0;
				z-index: 100;
				display: flex;
				width: 100%;
				height: 100%;
				.left {
					flex: 0 0 px2rem(100);
				}
				.center {
					flex: 1;
				}
				.right {
					flex: 0 0 px2rem(100);
				}
			}
		}

	}
</style>