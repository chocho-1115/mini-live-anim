<template>
	<view>
		<canvas id="likestar" class="canvas" type="2d" :width="realWidth" :height="realHeight" :style="{width: width + 'rpx', height: height + 'rpx'}"></canvas>
	</view>
</template>

<script>
	export default {
		name:"like-anim",
		props: {
			// 圆环进度百分比值
			like: {
				type: Number,
				default: 0,
				// 值大于0
				validator: val => {
					return val >= 0;
				}
			},
		},
		data() {
			return {
				realWidth: 300,
				realHeight: 600,
				width: 300,
				height: 600,
				
				likeImgList: [],
				canvas: null,
				ctx: null,
				aniFrameId: null,
				queue: {},
				
				iconWidth: 80,
				
			};
		},
		mounted() {
			this.init()
		},
		detached() {
		    const { realHeight, realWidth } = this;
		    if (this.aniFrameId) {
		      this.ctx.clearRect(0, 0, realWidth, realHeight);
		      this.canvas.cancelAnimationFrame(this.aniFrameId);
		      this.aniFrameId = null;
		    }
		    this.queue = {};
		},
		watch:{
			/**点赞个数变化 */
			like(newVal, oldVal) {
				// this.ctx初始化才能触发点赞个数变化
				if (this.ctx && newVal - oldVal > 0 && this.likeImgList.length) {
					// 自己点赞的时候，随机 1-3个气泡，im推送更新的时候，随机2-6个气泡
					const count =
						newVal - oldVal > 5 ? this.getRandomInt(2, 6) : this.getRandomInt(1, 3);
					this.likeClick(count);
				}
			}
		},
		methods:{
			init(){
				// 创建canvas上下文
				this.createSelectorQuery()
				  .select('#likestar')
				  .fields({ node: true })
				  .exec(res => {
					if (res[0]) {
					  const canvas = res[0].node;
					  this.canvas = canvas;
					  if (canvas.getContext) {
						this.ctx = canvas.getContext('2d');
						// 缩放canvs画布解决高清屏幕模糊问题
						const dpr = wx.getSystemInfoSync().pixelRatio;
						canvas.width = this.realWidth * dpr;
						canvas.height = this.realHeight * dpr;
						this.ctx.scale(dpr, dpr);
						
						let k = 0;
						for (let i = 0; i < 11; i++) {
						  const likeImgae = canvas.createImage();
						  likeImgae.src = `https://oss-magee.maijitv.com/maiji-mall/static/images/live/like-${i}.png`;
						  likeImgae.onload = () => {
							this.likeImgList.push(likeImgae);
							
							
							
							k++
							if(k==11) {
								setTimeout(()=>{
									console.log('=====')
									this.likeClick(10)
								},3000)
							}
						  };
						  
						}
					  }
					}
				  });
				
			},
			generatePathData() {
			    const { realWidth, realHeight } = this;
			    const p0 = {
			      x: 0.5 * realWidth,
			      y: realHeight,
			    };
			    const p1 = {
			      x: this.getRandom(-0.2, 0.5) * realWidth,
			      y: this.getRandom(0.6, 0.85) * realHeight,
			    };
			    const p2 = {
			      x: this.getRandom(0.7, 1) * realWidth,
			      y: this.getRandom(0.25, 0.5) * realHeight,
			    };
			    const p3 = {
			      x: this.getRandom(0.04, 0.7) * realWidth,
			      y: this.getRandom(0, 0.15) * realHeight,
			    };
			    return [p0, p1, p2, p3];
			},
			likeClick(count) {
			    const { length } = this.likeImgList;
			    const curId = new Date().getTime();
			
			    for (let i = 0; i < count; i++) {
					const image = this.likeImgList[this.getRandomInt(0, length - 1)];
					const anmationData = {
						id: curId + i,
						timer: 0, // 定时器
						opacity: 1, //透明度
						pathData: 1 // this.getRandomInt(0, 1)
						  ? this.generatePathData()
						  : this.generatePathDataReverse(), // 路径
						image: image,
						factor: {
						  speed: this.getRandom(0.01, 0.014), // 运动速度，值越小越慢
						  t: 0, //  贝塞尔函数系数
						},
						width: this.iconWidth * this.getRandom(0.9, 1.1),
					};
					if (Object.keys(this.queue).length > 0) {
						this.queue[anmationData.id] = anmationData;
					} else {
						this.queue[anmationData.id] = anmationData;
						this.aniFrameId = this.canvas.requestAnimationFrame(() => {
							this.bubbleAnimate();
						});
					}
			    }
			},
			bubbleAnimate() {
				
			    const { realHeight, realWidth } = this;
			    Object.keys(this.queue).forEach(key => {
					const anmationData = this.queue[+key];
					// console.log(anmationData)
					const { x, y } = this.updatePath(
						anmationData.pathData,
						anmationData.factor,
					);
					const { speed } = anmationData.factor;
					anmationData.factor.t += speed;
				
					let curWidth = anmationData.width;
					if (y > 0.25 * realHeight) {
						curWidth = (realHeight - y) / 2.5;
						curWidth = Math.min(anmationData.width, curWidth);
					} else {
						curWidth = (0.75 + y / realHeight) * anmationData.width;
					}

					let curAlpha = anmationData.opacity;
					curAlpha = y / realHeight;
					curAlpha = Math.min(1, curAlpha);
					this.ctx.globalAlpha = curAlpha;
					this.ctx.drawImage(
						anmationData.image,
						x - curWidth / 2,
						y,
						curWidth,
						curWidth,
					);
					// 贝塞尔曲线系数大于1，删除该气泡
					if (anmationData.factor.t > 1) {
						delete this.queue[anmationData.id];
					}
					if (y > realHeight) {
						delete this.queue[anmationData.id];
					}
					if (x < anmationData.width / 2) {
						delete this.queue[anmationData.id];
					}
					this.ctx.restore();
				});
				if (Object.keys(this.queue).length > 0) {
					// 每20ms刷新一次图层
					this.aniFrameId = this.canvas.requestAnimationFrame(() => {
						this.ctx.clearRect(0, 0, realWidth, realHeight);
						this.bubbleAnimate();
					});
				} else {
					this.ctx.clearRect(0, 0, realWidth, realHeight);
					this.canvas.cancelAnimationFrame(this.aniFrameId);
					this.aniFrameId = null;
				}
			},
			/**更新气泡的最新运动路径 */
			updatePath(data, factor) {
			    const p0 = data[0];
			    const p1 = data[1];
			    const p2 = data[2];
			    const p3 = data[3];
			
			    const { t } = factor;
			
			    /*贝塞尔曲线，计算多项式系数*/
			    const cx1 = 3 * (p1.x - p0.x);
			    const bx1 = 3 * (p2.x - p1.x) - cx1;
			    const ax1 = p3.x - p0.x - cx1 - bx1;
			
			    const cy1 = 3 * (p1.y - p0.y);
			    const by1 = 3 * (p2.y - p1.y) - cy1;
			    const ay1 = p3.y - p0.y - cy1 - by1;
			
			    const x = ax1 * (t * t * t) + bx1 * (t * t) + cx1 * t + p0.x;
			    const y = ay1 * (t * t * t) + by1 * (t * t) + cy1 * t + p0.y;
			    return {
					x,
					y,
			    };
			},
			getRandomInt(Min, Max){
				return ( Math.floor(Math.random()*(Max-Min+1)+Min) )
			},
			getRandom (Min, Max){ 
				return ( Min + Math.random()*(Max-Min) )
			},
			
		}
		
	}
</script>

<style lang="scss" scoped>
.canvas{
	background:rgba(0,0,0,.5);
}
</style>