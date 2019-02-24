<template>
	<div class="container">
		<p v-if="loadingTip!==''" class="loading-tip">{{ loadingTip }}</p>
		<img :src="loadingImage" class="loading-image"/>
	</div>
</template>

<script>
import loadingImage from './loading.gif';
export default {
	props: {
		tip: {
			type: String,
			default() {
				return ''
			}
		}
	},
	data() {
		return {
			loadingImage: loadingImage,
			loadingTip: this.tip
		}
	},
	watch: {
		tip: function(newString) {
			this.loadingTip = newString;
		}
	}
}
</script>

<style scoped>
.container {
	color: #212121;
	display: -webkit-box;
	display: -moz-box;
	height: 40px;
	overflow: hidden;
	-webkit-box-align: center;
	-moz-box-align: center;
	-webkit-box-pack:center;
	-moz-box-pack: center;
	width: 100%;
}
.loading-tip {
	font-size: 14px;
	float: left;
	line-height: 40px;
	margin: 0 10px 0 0;
	position: relative;
}
.loading-image {
	display: inline-block;
	float: left;
	height: 20px;
	position: relative;
	width: 20px;
}
</style>
