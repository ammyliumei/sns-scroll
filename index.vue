<template>
	<div ref="wrapper" class="scroller-container" :style="{top: top,bottom: bottom}">
		<div class="scroller-content">
			<div ref="scrollerList">
				<slot></slot>
			</div>
			<div v-if="pullUpLoad" class="scroller-pullup-wrapper">
				<loading v-if="isPullingUp" tip="正在加载"></loading>
				<p v-else class="scroller-before-pull-up">{{ pullUpText }}</p>
			</div>
		</div>
		<div v-if="pullDownRefresh" class="scroller-pulldown-wrapper" :style="pullDownStyle">
			<bubble v-if="beforePullDown" :y="bubbleY"></bubble>
			<div v-else class="pulldown-refresh-container">
				<loading v-if="isPullingDown" tip="正在刷新"></loading>
				<p v-else class="scroller-after-pull-down">{{ refreshText }}</p>
			</div>
		</div>
	</div>
</template>

<script>
import BScroll from 'better-scroll'
import bubble from './bubble'
import loading from './loading'
import getRect from './dom.js'
export default {
	props: {
		backgroundColor: {
			type: String,
			default() {
				return '#F5F5F5';
			}
		},
		top: {
			type: null,
			default() {
				return 0;
			}
		},
		bottom: {
			type: null,
			default() {
				return 0;
			}
		},
		probeType: {
			type: Number,
			default() {
				return 1;
			}
		},
		bindToWrapper: {
			type: Boolean,
			default() {
				return true;
			}
		},
		bounce: {
			type: Boolean,
			default() {
				return false;
			}
		},
		click: {
			type: null,
			default() {
				return true;
			}
		},
		listenBeforeScroll: {
			type: Boolean,
			default() {
				return true;
			}
		},
		scrollbar: {
			type: null,
			default() {
				return false;
			}
		},
		pullDownRefresh: {
			type: null,
			default() {
				return false;
			}
		},
		pullUpLoad: {
			type: null,
			default() {
				return false;
			}
		}
	},
	computed: {
		refreshText: function() {
			return this.pullDownRefresh && this.pullDownRefresh.text || '刷新成功';
		},
		pullUpText: function() {
			const moreText = this.pullUpLoad && this.pullUpLoad.text && this.pullUpLoad.text.more || '上拉加载更多';
			const noMoreTxt = this.pullUpLoad && this.pullUpLoad.text && this.pullUpLoad.text.noMore || '我是有底线的';
			return this.isAllowPullup?moreText:noMoreTxt;
		}
	},
	data() {
		return {
			bubbleY: 0,
			scroller: null,
			pullDownStyle: '',
			pullDownInitTop: -50,
			beforePullDown: true,
			isPullingDown: false,
			isPullingUp: false,
			isRebounding: false,
			isAllowPullup: true
		}
	},
	methods: {
		destroy: function() {
			this.scroller && this.scroller.destroy();
		},
		disable: function() {
			this.scroller && this.scroller.disable();
		},
		enable: function() {
			this.scroller && this.scroller.enable();
		},
		enablePullUp: function() {
			this.isAllowPullup = true;
		},
		disablePullUp: function() {
			this.isAllowPullup = false;
		},
		refresh: function() {
			this.scroller && this.scroller.refresh();
		},
		scrollTo: function(offsetX, offsetY, time) {
			this.scroller && this.scroller.scrollTo(offsetX, offsetY, time);
		},
		scrollToElement: function(el, time, offsetX, offsetY) {
			this.scroller && this.scroller.scrollToElement(el, time, offsetX, offsetY);
		},
		forceUpdate: function() {
			if(this.pullDownRefresh && this.isPullingDown) {
				this.isPullingDown = false;
				this._reboundPullDown().then(() => {
					this._afterPullDown();
				})
			}
			else if(this.pullUpLoad && this.isPullingUp) {
				this.isPullingUp = false;
				this.scroller.finishPullUp();
			}
			else {
				this.refresh();
			}
		},
		_initScroller: function() {
			if(!this.$refs.wrapper) { return; }
			if(this.$refs.scrollerList && (this.pullDownRefresh || this.pullUpLoad)) {
				this.$refs.scrollerList.style.minHeight = `${getRect(this.$refs.wrapper).height + 1}px`;
			}
			let options = {
				probeType: this.probeType,
				bounce: this.bounce,
				click: this.click,
				scrollbar: this.scrollbar,
				pullDownRefresh: this.pullDownRefresh?{
					threshold: 40,
					stop: 40,
					text: '刷新成功'
				}:this.pullDownRefresh,
				pullUpLoad: this.pullUpLoad?{
					threshold: 0,
					text: {
						more: '上拉加载更多',
						noMore: '我是有底线的'
					}
				}:this.pullUpLoad,
				bindToWrapper: this.bindToWrapper
			};
			this.scroller = new BScroll(this.$refs.wrapper, options);
			if(this.listenBeforeScroll) {
				this.scroller.on('beforeScrollStart', () => {
					document.activeElement.blur();
				})
			}
			if(this.pullDownRefresh) {
				this._initPullDownRefresh();
			}
			if(this.pullUpLoad) {
				this._initPullUpLoad();
			}
		},
		_initPullDownRefresh: function() {
			this.scroller.on('pullingDown', () => {
				this.beforePullDown = false;
				this.isPullingDown = true;
				setTimeout(() => {
					this.$emit('on-pulling-down');
				}, 500);
			});
			this.scroller.on('scroll', (pos) => {
				if(this.beforePullDown) {
					this.bubbleY = Math.max(0, pos.y + this.pullDownInitTop);
					this.pullDownStyle = `top:${Math.min(pos.y + this.pullDownInitTop, 0)}px`;
				}
				else {
					this.bubbleY = 0;
				}
				if(this.isRebounding) {
					this.pullDownStyle = `top:${0 - (this.pullDownRefresh.stop - pos.y)}px`;
				}
			})
		},
		_reboundPullDown: function() {
			const { stopTime = 600 } = this.pullDownRefresh;
			return new Promise((resolve) => {
				setTimeout(() => {
					this.isRebounding = true;
					this.scroller.finishPullDown();
					resolve();
				}, stopTime);
			})
		},
		_afterPullDown: function() {
			setTimeout(() => {
				this.pullDownStyle = `top:${this.pullDownInitTop}px`;
				this.beforePullDown = true;
				this.isRebounding = false;
				this.refresh();
			}, this.scroller.options.bounceTime);
		},
		_initPullUpLoad: function() {
			this.scroller.on('pullingUp', () => {
				if(this.isAllowPullup) {
					this.isPullingUp = true;
					setTimeout(() => {
						this.$emit('on-pulling-up');
					}, 2000);
				}
			})
		}
	},
	mounted() {
		this.$nextTick(() => {
			this._initScroller();
		})
	},
	components: {
		bubble,
		loading
	}
}
</script>

<style lang="less" scoped>
@import './index.less';
</style>
