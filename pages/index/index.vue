<template>
	<view class="page" v-show="showFadeIn" lazy-load="true">

		<!-- 画布 -->
		<canvas type="2d" id="cropperCanvas" canvas-id="cropperCanvas"
			:style="{'width':canvasWidth,'height':canvasWidth,'left':'9000px'}"></canvas>
		<!-- confetti画布 -->
		<view class="confetti" :style="{'marginTop':'-'+screenWidth*18/75+'px'}">
			<canvas id="c1" type="2d"
				:style="{'width':screenWidth*64/75+'px','height':screenWidth*64/75+'px','zIndex':confettiShow}"></canvas>
		</view>
		<!-- 旋转平台 -->
		<view class="stage" :style="{'transform':t}" v-show="showFadeIn" @touchstart="handleTouchstart"
			@touchend="handleTouchend">
			<view v-for="(item,index) in list" :key="index" class="page-item">
				<view class="img-wrap">
					<image :src="item.link"
						:style="{'filter':rm,'width':screenWidth*28.2/75+'px','height':screenWidth*28.2/75+'px'}"
						class="border" @click.stop="cropperFn" />
				</view>
			</view>
		</view>
		<!-- 主功能按钮(保存头像获取头像) -->
		<view class="btn-wrap" v-show="showFadeIn">
			<button @click.stop="cropperFn" hover-class="btn-hover" :style="{'width':screenWidth*9/26+'px','height':(screenWidth*5)/39+'px'}" v-show="!reuploadTag"></button>
			<button @click.stop="reupload" hover-class="btn-hover" :style="{'width':screenWidth*9/26+'px','height':(screenWidth*5)/39+'px'}" v-show="reuploadTag"></button>		
			<button @click.stop="saveAvatar" hover-class="btn-hover" :style="{'width':screenWidth*9/26+'px','height':(screenWidth*5)/39+'px'}"></button>
		</view>
		<!-- 头像栏 -->
		<view class="border-bar" v-show="showFadeIn"
			:style="{'height':screenHeight*90/812+'px','grid-template-columns':'repeat(auto-fill,'+screenHeight*90/812+'px)'}"
			@touchstart="handleTouchstart" @touchend="handleTouchend">
			<!-- 使用过滤数组，将list的第一个元素过滤掉，index+1 -->
			<view class="border-item" v-for="(item,index) in filterList" :key="index" @tap="selectborder(index+1)"
				:style="{'height':screenHeight*90/812+'px','width':screenHeight*90/812+'px'}">
				<image :src="item.link" mode="widthFix"
					:class="curBorderIdx==(index+1)? '.border-img-active':'.border-img'"></image> 
			</view>
		</view>

		<!-- 用户头像 -->
		<image :src="avatarUrl" :style="{'filter':rm,'width':screenWidth*28/75+'px','height':screenWidth*28/75+'px'}"
			class="usrAvatar"></image>
		<!-- 左右按钮滑动按钮 -->
		<view class="Prev_next" @click.stop="toPrev()">
			<image src="../../static/left.png" mode="widthFix" class="btn"
				:style="{'transform':'translate(-'+screenWidth*4/15+'px,'+screenWidth/5+'px)','-webkit-transform':'translate(-'+screenWidth*4/15+'px,'+screenWidth/5+'px)','width':screenWidth*4/25+'px'}">
			</image>
		</view>
		<view class="Prev_next" @click.stop="toNext()">
			<image src="../../static/right.png" mode="widthFix" class="btn"
				:style="{'transform':'translate('+screenWidth*4/15+'px,'+screenWidth/5+'px)','-webkit-transform':'translate('+screenWidth*4/15+'px,'+screenWidth/5+'px)','width':screenWidth*4/25+'px'}">
			</image>
		</view>
		<!-- 裁剪界面 -->
		<view v-show="cropperShow">
			<image-cropper id="image-cropper" limit_move="true" disable_rotate="true" :width="width" :height="height"
				:imgSrc="src" disable_ratio="true">
			</image-cropper>
			<!-- 按钮 -->
			<view class="cropper-button" style="z-index: 300;">
				<view class="button">
					<button @tap="bindCrop">确定</button>
				</view>
				<view class="button">
					<button @tap="bindCancelCrop">取消</button>
				</view>
			</view>
		</view>

		<!-- 顶部页眉 -->
		<view class="title-wrap" :style="{'top':titleHeight+'px'}" v-show="showFadeIn">
			<image src="../../static/footer.png" mode="widthFix" class="footer"></image>
			<image src="../../static/2023.png" mode="widthFix" class="title"></image>
		</view>

		<!-- 顶部装饰 -->
		<view class="ornament-top-wrap" @touchstart="handleTouchstart" @touchend="handleTouchend">
			<image src="../../static/bg1.png" mode="widthFix" class="ornament-top"></image>
		</view>

		<!-- 底部装饰 -->
		<view class="ornament-bottom-wrap">
			<image src="../../static/bg2.png" mode="widthFix" class="ornament-bottom"></image>
		</view>

		<!-- 背景 -->
		<image src="../../static/bg.png" mode="scaleToFill" class="bg-img" @touchstart="handleTouchstart"
			@touchend="handleTouchend"></image>

		<!-- 小青梅 -->
		<view class="mascot-wrap" @touchstart="knock">
			<image src="../../static/mascot.png" mode="widthFix" class="mascot"></image>
		</view>

		<!-- 点击祝福文字 -->
		<image :src="item.img" v-for="(item,index) in txtList" :key="item.id" mode="widthFix"
			:style="{'top':item.knockY+'px','left':item.knockX+'px'}" class="blessing"></image>

	</view>

</template>

<script>
	/* 引入裁剪组件 */
	import '../../wxcomponents/image-cropper/image-cropper';
	/* 引入confetti必要文件 */
	import lottie from '@/wxcomponents/confetti/lottie-miniprogram.js'
	import * as animationData from "../../wxcomponents/confetti/confetti.json"
	export default {
		data() {
			return {
				confettiShow: '-999', //是否显示confetti
				cropperShow: false, //是否显示裁剪图片
				width: 250, //裁剪功能显示宽度
				height: 250, //裁剪功能显示高度
				avatarUrl: null, //用户头像
				startTime: 0, //按下屏幕的时间
				startX: 0, //按下屏幕起始的坐标X
				startY: 0, //按下屏幕起始的坐标Y
				//翻转动画变量
				rotation: 0, //翻转角度
				rm: 'hue-rotate(0deg)',
				t: 'rotateZ(0deg)',
				curBorderIdx: -1, //当前选择的边框idx
				//画布大小
				canvasWidth: '300px',
				canvasHeight: '300px',
				titleHeight: null, //胶囊高度				
				screenHeight: null, //屏幕高度				
				screenWidth: null, //屏幕宽度				
				showFadeIn: false, // 初始加载小程序渐显动画				
				txtList: [], // 点击祝福文字列表
				num: 0, // 祝福文字累计点击
				blessing: null, // 祝福语图片路径
				// knock位置，显示祝福语的位置
				knockX: null,
				knockY: null,
				reuploadTag: false, // 重新上传标志
				// 边框
				list: [{
						link: '',
						id: "0",
						deg: 0
					},
					{
						link: '../../static/1.png',
						id: "1",
						deg: -60
					},
					{
						link: '../../static/2.png',
						id: "2",
						deg: -120
					},
					{
						link: '../../static/3.png',
						id: "3",
						deg: -180
					},
					{
						link: '../../static/4.png',
						id: "4",
						deg: -240
					}
				]
			}
		},
		computed: {
			//过滤数组
			filterList() {
				return this.list.filter((data) => {
					return data.id !== "0" // 把边框list数组的id为0的元素空出来，使得一开始界面没有边框
				})
			}
		},
		onLoad() {
			//初始化画布
			this.$data.cropper = this.selectComponent("#image-cropper");
			// 获取胶囊位置
			let res = uni.getMenuButtonBoundingClientRect()
			this.titleHeight = res.top //胶囊距离屏幕顶部距离
			console.log("this.titleHeight", this.titleHeight)
			// 屏幕信息
			uni.getSystemInfo({
				success: (res) => {
					this.$data.ratio = res.windowWidth / 750
					this.screenHeight = res.screenHeight //屏幕高度
					this.screenWidth = res.screenWidth //屏幕高度
					console.log('screenWidth', this.screenWidth)
				}
			})
		},
		onReady() {
			// 屏幕信息再次获取，防止获取不到
			uni.getSystemInfo({
				success: (res) => {
					// console.log('系统信息', res)
					this.$data.ratio = res.windowWidth / 750 // 微信默认750屏宽比例
					this.screenHeight = res.screenHeight //屏幕高度
				}
			})
			setTimeout(() => {
				this.showFadeIn = true
			}, 150)
			// 初始化祝福文字列表
			this.txtList = []
			this.num = 0
		},
		onShow() {
			// 初始化祝福文字列表
			this.txtList = []
			this.num = 0
			// 重新上传清零
			this.reuploadTag = false 
		},
		methods: {
			// confetti
			init() {
				let that = this
				uni.createSelectorQuery().selectAll('#c1').node(res => {
					const canvas = res[0].node
					const context = canvas.getContext('2d')

					canvas.width = that.screenWidth * 64 / 75
					canvas.height = that.screenWidth * 64 / 75

					lottie.setup(canvas)
					this.ani = lottie.loadAnimation({
						loop: false,
						autoplay: true,
						animationData: animationData,
						rendererSettings: {
							context,
						},
					})
					this._inited = true
				}).exec()
			},
			play() {
				this.ani.play()
			},
			pause() {
				this.ani.pause()
			},
			// 敲击上升祝福语
			knock(e) {
				let that = this
				let knockX = e.changedTouches[0].clientX; //点击坐标X
				let knockY = e.changedTouches[0].clientY; //点击坐标Y
				this.num += 1
				const index = Math.floor(Math.random() * 10 + 1)
				let list = JSON.parse(JSON.stringify(this.txtList))
				list.push({
					img: `../../static/blessing/blessing${index}.png`,
					id: this.num,
					knockX: knockX,
					knockY: knockY
				})
				// 数组长度阈值，超过此阈值直接改变长度，避免卡顿
				let max = 15
				if (list.length === max) {
					list.reverse()
					list.length = list.length - (max - 5)
					list.reverse()
				}
				this.txtList = list
				console.log(this.num, this.txtList)
			},
			// 裁剪界面
			cropperFn() {
				let that = this
				this.cropperShow = true; //显示裁剪界面
				this.cropperShowGlobal = true
				if(this.avatarUrl == null){
					that.$data.cropper.upload()
					this.reuploadTag = true
				} 
				if (this.cropperShowGlobal == false) {
					this.cropperShow = false
				}
			},
			// 重新上传图片
			reupload(){
				let that = this
				this.cropperShow = true; //显示裁剪界面
				this.cropperShowGlobal = true
				that.$data.cropper.upload() //打开上传图片
				if (this.cropperShowGlobal == false) {
					this.cropperShow = false
				}
			},
			// 确认裁剪
			bindCrop() {
				let that = this
				that.$data.cropper.getImg(res => {
					that.cropperShow = false
					this.cropperShowGlobal = false
					that.avatarUrl = res.url // 替换原本的图片
				})
			},
			// 取消裁剪
			bindCancelCrop() {
				this.cropperShow = false
				this.cropperShowGlobal = false
			},
			// 边框选择
			selectborder(index) {
				this.curBorderIdx = index // 当前选择边框序号
				this.rotation = this.list[index].deg //选装角度
				this.rm = 'hue-rotate(-' + this.rotation.toString() + 'deg)'
				this.t = 'rotateZ(' + this.rotation.toString() + 'deg)'
			},
			// 向左滑动
			toPrev() {
				//容错，防止向左滑动
				if (this.curBorderIdx == 1 || this.curBorderIdx == 0) {
					console.log('左边没有啦')
					uni.showToast({
						icon: 'none',
						title: '左边没有啦'
					})
				} else { //向左滑动，
					this.curBorderIdx -= 1 //当前指向的边框数组下标-1
					this.rotation = this.list[this.curBorderIdx].deg
					this.rm = 'hue-rotate(-' + this.rotation.toString() + 'deg)'
					this.t = 'rotateZ(' + this.rotation.toString() + 'deg)'
				}

			},
			// 向右滑动
			toNext() {
				//容错，防止向右滑动
				if (this.curBorderIdx == 4) {
					console.log('右边没有啦')
					uni.showToast({
						icon: 'none',
						title: '右边没有啦'
					})
				} else {
					this.curBorderIdx += 1 //当前指向的边框数组下标+1
					this.rotation = this.list[this.curBorderIdx].deg
					this.rm = 'hue-rotate(-' + this.rotation.toString() + 'deg)'
					this.t = 'rotateZ(' + this.rotation.toString() + 'deg)'
				}

			},
			//按下屏幕
			handleTouchstart(e) {
				//记录用户按下的时间
				this.startTime = e.timeStamp
				//记录用户按下的坐标
				this.startX = e.changedTouches[0].clientX;
				this.startY = e.changedTouches[0].clientY;
			},
			//离开屏幕
			handleTouchend(e) {
				var endX = e.changedTouches[0].clientX;
				var endY = e.changedTouches[0].clientY;
				//判断按下的时长，超过设置的按下最长时间，将不被判断为滑动操作
				if (e.timeStamp - this.startTime > 3000) {
					return;
				}
				//滑动的方向
				let direction = "";
				//先判断用户滑动的距离  用[绝对值] 是否合法  合法：判断滑动的方向
				if (Math.abs(endX - this.startX) > 10 && Math.abs(endY - this.startY) < 200) {
					//滑动方向
					direction = endX - this.startX > 0 ? "right" : "left";
				} else {
					return;
				}
				console.log('direction', direction);
				if (direction === 'right') { //判断为向右滑动，则选择边框向前移动
					this.toPrev()
				} else if (direction === 'left') { //判断为向左滑动，则选择边框向后移动
					this.toNext()
				}
			},
			// 保存头像
			saveAvatar() {
				if (this.avatarUrl == null || this.avatarUrl == '') {
					uni.showToast({
						title: '请先点击获取头像',
						icon: 'none',
						duration: 2000
					})
				} else if (this.curBorderIdx == -1) { //初始加载小程序没有显示边框，将其设置为-1
					uni.showToast({
						title: '请选择一个边框',
						icon: 'none'
					})
				} else {
					uni.showLoading({
						title: '正在生成头像',
						mask: true
					})
					this.drawImg() //调用图片合成函数
					
				}
			},
			// 合成图片函数
			drawImg() {
				const that = this
				// 选中合成画布cropperCanvas
				uni.createSelectorQuery().select('#cropperCanvas').fields({
					node: true,
					size: true
				}).exec(res => {
					that.$data.canvas = res[0].node
					that.$data.ctx = that.$data.canvas.getContext('2d')
					//获取背景图片
					let p1 = new Promise(function(resolve, reject) {
						//将用户上传的临时文件缓存下来，才可以在画布上进行绘制
						uni.getImageInfo({
							src: that.avatarUrl,
							success: (res) => resolve(res),
							fail: (res) => reject(res)
						})
					})
					//获取边框图片
					let p2 = new Promise(function(resolve, reject) {
						//下载用户选择的边框
						uni.getImageInfo({
							src: `../../static/${that.curBorderIdx}.png`,
							success: (res) => resolve(res),
							fail: (res) => reject(res)
						})


					})
					Promise.all([p1, p2]).then(
						res => {
							//设置画布大小
							that.$data.canvas.width = res[0].width / that.$data.ratio
							that.$data.canvas.height = res[0].height / that.$data.ratio
							that.$data.canvasWidth = res[0].height + 'px'
							that.$data.canvasHeight = res[0].height + 'px'
							const backImg = that.$data.canvas.createImage()
							backImg.src = res[0].path
							let borderPath = res[1].path
							backImg.onload = () => {
								const Img = that.$data.canvas.createImage()
								Img.src = '../../' + borderPath
								Img.onload = () => {
									that.$data.ctx.drawImage(backImg, 0, 0, that.$data.canvas.width,
										that.$data.canvas.height); //绘制背景
									that.$data.ctx.drawImage(Img, 0, 0, that.$data.canvas.width, that
										.$data.canvas.height); //绘制边框
									// 输出成图片，解决性能故障
									// 绘制最终展示图片时的错误次数
									var canvasToImageFileError = 1;
									// 绘制最终展示图片时的延时
									var canvasToImageFileDelayTime = 200;
									// 绘制最终展示图片时的画质
									var canvasToImageFileQuality = 1;
									let _this = this
									// 若用户手机性能差则回调此canvasToImageFile()
									function canvasToImageFile(_this) {
										return new Promise(async (resolve, reject) => {
											setTimeout(() => {
												// 输出成图片
												uni.canvasToTempFilePath({
													x: 0,
													y: 0,
													width: _this.$data.canvas.width,
													height: _this.$data.canvas.height,
													destWidth: _this.$data.canvas.width *2, //*2提高输出图片的质量
													destHeight: _this.$data.canvas.height * 2,
													canvas: _this.$data.canvas,
													quality: canvasToImageFileQuality, //图片输出质量
													success: (res) => {
														console.log(
															'√生成成功，尝试次数:' +
															canvasToImageFileError +
															',延迟:' +
															canvasToImageFileDelayTime +
															',画质:' +
															canvasToImageFileQuality,
															res);
														console.log(
															'[生成图片路径]',
															res
															.tempFilePath
														);
														uni.hideLoading()
														resolve(res
															.tempFilePath
														)
													},
													fail: (e) => {
														uni.hideLoading()
														if (canvasToImageFileError <=
															5) {
															// 错误次数+1
															canvasToImageFileError
																+= 1;
															// 延迟+100
															canvasToImageFileDelayTime
																+= 100;
															// 画质减去0.1
															canvasToImageFileQuality
																-= 0.1;
															// 回调
															canvasToImageFile
																(
																	_this
																);
														} else {
															console
																.log(
																	'×生成失败，尝试次数:' +
																	canvasToImageFileError +
																	',延迟:' +
																	canvasToImageFileDelayTime +
																	',画质:' +
																	canvasToImageFileQuality,
																	e);
															reject(
																'绘制失败'
															);
															uni.showToast({
																mask: true,
																icon: 'none',
																title: '绘制失败，请重新上传'
															});
														}
													}
												})
											}, canvasToImageFileDelayTime)
										})
									}
									// 调用canvas2Img函数
									canvasToImageFile(_this).then(function(res) {
										uni.hideLoading()
										uni.saveImageToPhotosAlbum({
											filePath: res,
											success: (res) => {
												_this.confettiShow = '999'
												_this.init()
												uni.showToast({
													title: '已保存到手机相册',
													icon: 'none'
												})
												setTimeout(()=>{
													_this.confettiShow = '-999'
												},2000)
											},
											fail: (err) => {
												uni.showToast({
													title: '取消保存',
													icon: 'none'
												})
											}
										})
									}).catch(function(e) {
										uni.showToast({
											title: e,
											icon: 'none'
										})
									})
								}
							}
						}
					)
				})
			},

		}
	}
</script>

<style lang="less">
	@import url("index.less");
</style>
