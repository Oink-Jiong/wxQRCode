<template>
	<view class="container">

		<!-- 先绘制二维码 -->
		<canvas :style="{ width: qrcode_w + 'px', height: qrcode_h + 'px', position: 'absolute', left: 0, bottom: '-' + qrcode_h + 'px' }"
		 canvas-id="myCanvas" class="my-canvas"></canvas>

		<!-- 绘制完整海报 -->
		<canvas :style="{ width: qrcode_w2 + 'px', height: qrcode_h2 + 'px', position: 'absolute', left: 0, bottom: '-' + qrcode_h2 + 'px'  }"
		 canvas-id="myQrcode"></canvas>

	</view>
</template>

<script>
	import QRCode from '@/utils/weapp.qrcode.min.js';

	export default {

		data() {
			return {
				// 绘制二维码的canvas
				qrcode_w: 300,
				qrcode_h: 300,

				// 绘制所有信息的canvas
				qrcode_w2: 300,
				qrcode_h2: 400,
			}
		},

		onShow() {
			// 二维码画布大小
			this.qrcode_w = this.getPxToRpx(600);
			this.qrcode_h = this.getPxToRpx(600);

			// 所有信息画布大小
			this.qrcode_w2 = this.getPxToRpx(600);
			this.qrcode_h2 = this.getPxToRpx(800);

			this.drawQrCode()
		},

		methods: {

			// px单位转换rpx，宽度（高度可能存在问题）
			getPxToRpx(pxParam) {
				const W = wx.getSystemInfoSync().windowWidth;
				const rate = 750.0 / W;
				return pxParam / rate;
			},

			// 绘制二维码
			drawQrCode() {
				new QRCode('myCanvas', {
					// colorLight: '#ffcc66',
					// colorDark: "#ffcc66",
					width: this.qrcode_w,
					height: this.qrcode_h,
					text: `https://www.baidu.com/`,
					padding: this.getPxToRpx(100), // 生成二维码四周自动留边宽度，不传入默认为0
					correctLevel: QRCode.CorrectLevel.L, // 二维码可辨识度
					callback: (res) => {
						// 接下来就可以直接调用微信小程序的api保存到本地或者将这张二维码直接画在海报上面去，看各自需求
						console.log('二维码： -> ' + res.path)
						this.drawAll(res.path)
					}
				})
			},

			// 绘制完整海报
			drawAll(qrcodePath) {

				// canvas创建画图
				const ctx = wx.createCanvasContext('myQrcode');
				// 绘制背景图
				ctx.setFillStyle('#fff')
				ctx.fillRect(0, 0, this.qrcode_w2, this.qrcode_h2)
				// 绘制二维码
				ctx.drawImage(qrcodePath, 0, 0, this.qrcode_w, this.qrcode_h);
				// 绘制字体，颜色，与位置
				ctx.setFontSize(14)
				ctx.setFillStyle('#474747')
				ctx.setTextAlign('center')
				ctx.fillText('测试文字', this.getPxToRpx(300), this.getPxToRpx(620))
				ctx.fillText('yyyy-MM-dd HH:mm:ss', this.getPxToRpx(300), this.getPxToRpx(680))

				// canvasToTempFilePath必须要在draw的回调中执行，否则会生成失败，官方文档有说明
				ctx.draw(false, setTimeout(() => {
					uni.canvasToTempFilePath({
						x: 0,
						y: 0,
						width: this.qrcode_w2,
						height: this.qrcode_h2,
						destWidth: this.qrcode_w2 * 2,
						destHeight: this.qrcode_h2 * 2,
						canvasId: 'myQrcode',
						success: (res) => {
							// 绘制成功后，使用预览图片api，展示二维码海报
							console.log('生成最终海报： -> ' + res.tempFilePath);
							uni.hideLoading();
							uni.previewImage({
								current: res.tempFilePath, // 当前显示图片的http链接
								urls: [res.tempFilePath]
							})
						},
						fail: (err) => {
							console.log(err);
							uni.hideLoading();
							uni.showToast({
								title: '生成失败',
								icon: "none"
							})
						}
					})
				}, 200));
			}

		}
	}
</script>

<style lang="scss" scoped>
	.container {
		overflow: hidden;
	}

	// 画二维码的canvas，定位到看不到的位置
	.my-canvas {
		position: absolute;
		bottom: 0;
		left: 0;
		right: 0;
	}
</style>
