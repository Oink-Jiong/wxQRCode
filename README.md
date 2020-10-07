# 微信小程序canvas自定义分享二维码

--------------------------------------------------

## 目标
- 应用uni-app框架（或者原生微信小程序同理）
- 使用 `Canvas` 绘制自定义的二维码、文字画布
- 将画布转化成可供 `分享保存` 的图片

--------------------------------------------------

## 解决方案

### 方案1
1. 定义一个二维码临时canvas，如 `canvas-id="myCanvas"`； 以及 绘制完整海报canvas，如`canvas-id="myQrcode"`
``` html
	<!-- 先绘制二维码 -->
	<canvas :style="{ width: qrcode_w + 'px', height: qrcode_h + 'px', position: 'absolute', left: 0, bottom: '-' + qrcode_h + 'px' }"
	 canvas-id="myCanvas" class="my-canvas"></canvas>

	<!-- 绘制所有内容 -->
	<canvas :style="{ width: qrcode_w2 + 'px', height: qrcode_h2 + 'px', position: 'absolute', left: 0, bottom: '-' + qrcode_h2 + 'px'  }"
	 canvas-id="myQrcode"></canvas>

```
2. 使用封装过的 `weapp.qrcode.min.js` 进行生成动态二维码，并在回调函数 callback 获取图片资源的网络路径
``` javaScript
import QRCode from '@/utils/weapp.qrcode.min.js';
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
}
```
3. 运用微信小程序api `CanvasContext.drawImage`，将上述2中的网络路径的图片资源，绘制图像到画布2中：`myQrcode`
``` javaScript
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
```
- [GitHub地址：https://github.com/Jiji-Jiong/wxQRCode](https://github.com/Jiji-Jiong/wxQRCode)


### 方案2（只演示方案1，可做训练）
1. 定义一个二维码临时canvas，如 `canvas-id="myCanvas"`； 以及 绘制完整海报canvas，如`canvas-id="myQrcode"`
2. 使用 `weapp.qrcode.min.js` 进行在 画布1 `myCanvas` 生成动态二维码
3. 使用 `wx.canvasToTempFilePath` ，将画布1 `myCanvas` 中的二维码导出来，生成 res.tempFilePath 网络路径的图片资源
4. 运用微信小程序api `CanvasContext.drawImage`，将上述2中的网络路径的图片资源，绘制图像到画布2中：`myQrcode`

--------------------------------------------------

## 提示
1. canvas可以使用`绝对定位`设置到界面之外，控制隐藏显示（根据自身需要）
2. canvas在绘制draw的时候，不可以设置 `display: none` 的隐藏效果，否则绘制不出（和Echarts图表类库同理）"# wxQRCode" 
