better-scroll 是一款重点解决移动端（已支持 PC）各种滚动场景需求的插件。它的核心是借鉴的 iscroll 的实现，它的 API 设计基本兼容 iscroll，在 iscroll 的基础上又扩展了一些 feature 以及做了一些性能优化。

better-scroll 是基于原生 JS 实现的，不依赖任何框架。它编译后的代码大小是 63kb，压缩后是 35kb，gzip 后仅有 9kb，是一款非常轻量的 JS lib。

better-scroll 托管在 Npm 上，执行如下命令安装：

npm install better-scroll --save

接下来就可以在代码中引入了，webpack 等构建工具都支持从 node_modules 里引入代码：

import BScroll from 'better-scroll'

如果是 ES5 的语法，如下：

var BScroll = require('better-scroll')

better-scroll 也支持直接用 script 加载的方式，加载后会在 window 上挂载一个 BScroll 的对象。

你可以直接用：https://unpkg.com/better-scroll@1.0.1/dist/bscroll.min.js 这个地址。也可以把 dist 目录下的文件拷贝出去发布到自己的 cdn 服务器。

docs   https://ustbhuangyi.github.io/better-scroll/#/

组件可以传递的参数如下：

背景色:  :backgroundColor="#fff000"或者不传，默认:#F5F5F5

距离顶部高度:   top="100px"，默认为：0

距离底部高度:   bottom="40px"，默认为：0

是否启用弹性模式：  :bounce="true"，默认为：false

是否显示滚动条：  :scrollbar="true"，默认为：false

要启用下拉刷新的功能，请传递配置对象：  :pullDownRefresh="{threshold: 40, stop: 40, text: '刷新成功'}"

其中threshold代表回弹的位置,stop代表停止的位置，如果不想启用，可以不传

要启用上拉加载的功能，请传递配置对象：  :pullUpLoad="{threshold: 0, text: { more: '上拉加载更多', noMore: '我是有底线的' } }"

组件可以调用的API如下：

销毁滚动组件: `this.$refs.scroll && this.$refs.scroll.destroy();`

禁用滚动组件：`this.$refs.scroll && this.$refs.scroll.disable();`

启用滚动组件：`this.$refs.scroll && this.$refs.scroll.enable();`

启用上拉加载功能：`this.$refs.scroll && this.$refs.scroll.enablePullUp()`;

禁用上拉加载功能: `this.$refs.scroll && this.$refs.scroll.disablePullUp();`

刷新滚动组件，防止在滚动组件中动态添加内容后，滚动组件的高度没有重置.

不过组价内部已经设置了内容高度监听，会自动刷新出高度，不用过度担心，不过还用要说一下调用方法:

`this.$refs.scroll && this.$refs.scroll.refresh();`

让滚动组件滚动到某个位置: `this.$refs.scroll && this.$refs.scroll.scrollTo(offsetX,offsetY,time);`

一般来说垂直的滚动组件是要设置 offsetX = 0; offsetY为你需要的高度; time为滚动事件

滚动到某个元素位置: `this.$refs.scroll && this.$refs.scroll.scrollToElement(el, time, offsetX, offsetY);`

组件下拉刷新和上拉加载使用方式如下：

```
<template>
	<div id="app" class="app-container">
		<app-scroller
			ref="scroll"
			:bounce="true"
			:scrollbar="true"
			:pullDownRefresh="pullDownRefresh"
			:pullUpLoad="pullUpLoad"
		    @on-pulling-down="afterPullDown"
		    @on-pulling-up="afterPullUp">
			<div>
				<div v-for="(item,index) in listNum" :key="index">
					滚动组件里面的内容
				</div>
			</div>
		</app-scroller>
	</div>
</template>

<script>
import AppScroller from "@/components/common/AppScroller"
export default {
	data() {
		return {
			pullDownRefresh: {
				threshold: 40,
				stop: 40,
				text: '刷新成功'
			},
			pullUpLoad: {
				threshold: 0,
				text: {
					more: '上拉加载更多',
					noMore: '我是有底线的'
				}
			},
			listNum: 30
		}
	},
	methods: {
		//下拉刷新，手指touch结束的时候触发的事件
		afterPullDown() {
			this.listNum = 30;
			//完成下拉刷新
			this.$refs.scroll && this.$refs.scroll.forceUpdate();
		},
		//上拉加载，手指touch结束时触发的事件
		afterPullUp() {
			this.listNum = this.listNum + 30;
			//完成上拉加载
			this.$refs.scroll && this.$refs.scroll.forceUpdate();
			//禁用上拉加载
			this.$refs.scroll && this.$refs.scroll.disablePullUp();
		}
	},
	created() {

	},
	components: {
		AppScroller
	}
};
</script>

<style lang="less" scoped>
.app-container {
	height: 100%;
	position: relative;
	width: 100%;
}
</style>
```
