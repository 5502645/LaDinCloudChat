<template>
	<view>
		<view>
			<view class="toolbar">
				<view class="iconfont icon-bold" @click="toolBarClick('bold')"></view>
				<view class="iconfont icon-italic" @click="toolBarClick('italic')"></view>
				<view class="iconfont icon-xiahuaxian1" @click="toolBarClick('header')"></view>
				<view class="iconfont icon-underline" @click="toolBarClick('underline')"></view>
				<view class="iconfont icon-strike" @click="toolBarClick('strike')"></view>
				<view class="iconfont icon-sup" @click="toolBarClick('sup')"></view>
				<view class="iconfont icon-sub" @click="toolBarClick('sub')"></view>
				<view class="iconfont icon-alignleft" @click="toolBarClick('alignleft')"></view>
				<view class="iconfont icon-aligncenter" @click="toolBarClick('aligncenter')"></view>
				<view class="iconfont icon-alignright" @click="toolBarClick('alignright')"></view>
				<view class="iconfont icon-link" @click="toolBarClick('link')"></view>
				<view class="iconfont icon-image" @click="toolBarClick('imgage')"></view>
				<view class="iconfont icon-code" @click="toolBarClick('code')"></view>
				<view class="iconfont icon-table" @click="toolBarClick('table')"></view>
				<view class="iconfont icon-qingkong" @click="toolBarClick('clear')"></view>
			</view>
			<view class="input-content">
				<textarea auto-height maxlength="-1" v-model="textareaDataSync" @blur="getCursor"></textarea>
			</view>
		</view>
		<view class="preview" v-if="showPreview && textareaHtmlSync">
			<scroll-view scroll-y :style="'height:'+screenHeight/2.5+'px;padding:10px;box-sizing: border-box;'">
				<uParse :content="textareaHtmlSync" @preview="preview" @navigate="navigate" />
			</scroll-view>
		</view>
	</view>
</template>

<script>
	import marked from '../marked'
	import uParse from '../uParse/src/wxParse.vue'
	export default {
		name: "ly-markdown",
		components: {
			uParse
		},
		data: function() {
			return {
				screenHeight: 0,
				cursor: 0,
				textareaDataSync: '',
				textareaHtmlSync: ''
			}
		},
		props: {
			textareaData: {
				type: String,
				default: ''
			},
			textareaHtml: {
				type: String,
				default: ''
			},
			showPreview: {
				type: Boolean,
				default: false
			}
		},
		methods: {
			preview(src, e) {
				uni.previewImage({
					urls: src,
				})
			},
			navigate(href, e) {
				// ???????????????????????????????????????????????????????????????????????????href?????????????????????webview?????????????????????????????????
				// #ifdef APP-PLUS
				plus.runtime.openURL(href)
				// #endif
				// #ifdef MP-WEIXIN
				uni.setClipboardData({
					data: href,
					success: function() {
						uni.showModal({
							content: "???????????????,??????????????????????????????",
							showCancel: false
						})
					}
				})
				// #endif
			},
			toolBarClick(type) {
				if (type == 'bold') {
					this.textareaDataSync += "**????????????** "
				} else if (type == "italic") {
					this.textareaDataSync += "*??????* "
				} else if (type == "header") {
					uni.showActionSheet({
						itemList: ["??????1", "??????2", "??????3", "??????4", "??????5", "??????6"],
						success: res => {
							switch (res.tapIndex) {
								case 0:
									this.textareaDataSync += "# ??????1\r";
									return;
								case 1:
									this.textareaDataSync += "## ??????2\r";
									return;
								case 2:
									this.textareaDataSync += "### ??????3\r";
									return;
								case 3:
									this.textareaDataSync += "#### ??????4\r";
									return;
								case 4:
									this.textareaDataSync += "##### ??????5\r";
									return;
								case 5:
									this.textareaDataSync += "###### ??????6\r";
									return;
							}
						}
					})
				} else if (type == "underline") {
					this.textareaDataSync += "++?????????++ "
				} else if (type == "strike") {
					this.textareaDataSync += "~~?????????~~ "
				} else if (type == "sup") {
					this.textareaDataSync += "^?????????^ "
				} else if (type == "sub") {
					this.textareaDataSync += "~?????????~ "
				} else if (type == "alignleft") {
					this.textareaDataSync += "\n::: hljs-left\n\n?????????\n\n:::\n"
				} else if (type == "aligncenter") {
					this.textareaDataSync += "\n::: hljs-center\n\n????????????\n\n:::\n"
				} else if (type == "alignright") {
					this.textareaDataSync += "\n::: hljs-right\n\n\n\n?????????\n\n:::\n"
				} else if (type == "link") {
					this.textareaDataSync += "[????????????????????????](??????????????????) "
				} else if (type == "imgage") {
					this.textareaDataSync += "![](????????????????????????) "
				} else if (type == "code") {
					this.textareaDataSync += "\n``` ????????? \n\n```\n"
				} else if (type == "table") {
					this.textareaDataSync += "\n|???1|???2|???3|\n|-|-|-|\n|?????????1|?????????2|?????????3|\n"
				} else if (type == "clear") {
					uni.showModal({
						title: "??????",
						content: "?????????????",
						success: res => {
							if (res.confirm) {
								this.textareaDataSync = "";
							}
						}
					})
				}
			},
			getCursor(e) {
				//??????????????????????????????cursor??????,????????????
				//this.cursor = e.detail.cursor; 
			}
		},
		watch: {
			"textareaDataSync": function(newValue, oldValue) {
				this.textareaHtmlSync = marked(newValue)
				this.$emit('update:textareaData', newValue)
				this.$emit('update:textareaHtml', this.textareaHtmlSync)
			}
		},
		created() {
			this.textareaDataSync = this.textareaData;
			this.textareaHtmlSync = this.textareaHtml;
		},
		mounted() {
			uni.getSystemInfo({
				success: res => {
					this.screenHeight = res.screenHeight
				}
			})
		}
	}
</script>

<style>
	@import './markdown.css';
	@import url("../uParse/src/wxParse.css");

	.input-content {
		width: 100%;
	}

	.input-content textarea {
		padding: 16upx 25upx 15upx 25upx;
		font-size: 30upx;
		min-height: 500upx;
		line-height: 1.5;
	}

	.preview {
		border-top: 1upx solid #e0e0e0;
		width: 100%;
	}

	.toolbar {
		width: 100%;
		border: none;
		box-shadow: 0 0upx 4upx rgba(0, 0, 0, 0.157), 0 0upx 4upx rgba(0, 0, 0, 0.227);
	}

	.toolbar .iconfont {
		display: inline-block;
		cursor: pointer;
		height: 61.6upx;
		width: 61.6upx;
		margin: 13upx 0 11upx 0upx;
		font-size: 33upx;
		padding: 10upx 13upx 11upx 8upx;
		color: #757575;
		border-radius: 11upx;
		text-align: center;
		background: none;
		border: none;
		outline: none;
		line-height: 2.2;
		vertical-align: middle;
	}

	.input-content {
		min-height: ;
	}
</style>
