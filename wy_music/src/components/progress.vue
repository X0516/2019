<template>
    <!--进度条拖动-->
	<div class="mmProgress" ref="mmProgress" @click="barClick">
        <div class="mmProgress-bar" ref="mmProgressBar"  ></div>
        <div class="mmProgress-outer" ref="mmPercentProgress"></div>
        <div class="mmProgress-inner" ref="mmProgressInner">
            <div class="mmProgress-dot" @mousedown="barDown" @touchstart.prevent="barDown"></div>
        </div>
	</div>
</template>

<script>
    const dotWidth = 10;
	export default {
		name: "mmProgress",
        data(){
		    return {
                playpercent:0,
                move: {
                    status: false, // 是否可拖动
                    startX: 0, // 记录最开始点击的X坐标
                    left: 0, // 记录当前已经移动的距离
                }
            }
        },
        props: {
		    // 进度值一
            percent: {
                // type: [Number],
                default: 0
            },
            // 进度值二（歌曲缓冲进度用）
            percentProgress: {
                // type: [Number],
                default: 0
            }
        },
        mounted(){
            this.$nextTick(() => {
                this.bindEvents();
                const barWidth = this.$refs.mmProgress.clientWidth - dotWidth;
                const offsetWidth = this.percent * barWidth;
                this.moveSilde(offsetWidth)
            })
        },
        methods:{
		    //添加绑定事件
            bindEvents(){
                document.addEventListener('mousemove',this.barMove);
                document.addEventListener('mouseup',this.barUp);
                
                document.addEventListener('touchmove',this.barMove);
                document.addEventListener('touchend',this.barUp)
            },
            //移除绑定事件
            unbindEvents(){
                document.removeEventListener('mousemove',this.barMove);
                document.removeEventListener('mouseup',this.barUp);
                
                document.removeEventListener('touchmove',this.barMove);
                document.removeEventListener('touchend',this.barUp)
            },
		    //点击事件
            barClick(e){
                let rect = this.$refs.mmProgress.getBoundingClientRect();
                let offsetWidth = Math.min(this.$refs.mmProgress.clientWidth - dotWidth,Math.max(0,e.clientX - rect.left));
                this.moveSilde(offsetWidth);
                this.commitPercent()
            },
            //鼠标按下事件
            barDown(e){
                this.move.status = true;
                this.move.startX = e.clientX || e.touches[0].pageX;
                this.move.left = this.$refs.mmProgressInner.clientWidth
            },
            //鼠标/触摸移动事件
            barMove(e){
                if (!this.move.status) {
                    return false
                }
                let endX = e.clientX || e.touches[0].pageX;
                let dist = endX - this.move.startX;
                let offsetWidth = Math.min(this.$refs.mmProgress.clientWidth - dotWidth,Math.max(0,this.move.left + dist));
                this.moveSilde(offsetWidth);
                this.commitPercent()
            },
            //鼠标/触摸释放事件
            barUp(e){
                e.stopPropagation();
                this.move.status = false;
                //this.commitPercent()
            },
            //移动滑块
            moveSilde(offsetWidth){
                this.$refs.mmProgressInner.style.width =this.$refs.mmProgress.offsetWidth*this.playpercent+"px";
                this.playpercent=((offsetWidth)/(document.documentElement.offsetWidth*0.73)).toFixed(2)
                // console.log(this.playpercent);
                // this.$emit(ProgressChange,this.playpercent)
                // console.log(this.$refs.mmProgress.offsetWidth);
                
                
            },
            //修改percent
            commitPercent(){
                let lineWidth = this.$refs.mmProgress.clientWidth - dotWidth;
                // let percent = this.$refs.mmProgressInner.clientWidth / lineWidth;
                this.$emit('percentChange', this.playpercent)
            }
        },
        watch:{
            percent(newPercent) {
                if (newPercent >= 0 && !this.move.status) {
                    const barWidth = this.$refs.mmProgress.clientWidth - dotWidth;
                    const offsetWidth = newPercent ;
                    this.moveSilde(offsetWidth)
                }
            },
            percentProgress(newValue){
                let offsetWidth = this.$refs.mmProgress.clientWidth * newValue;
                this.$refs.mmPercentProgress.style.width = `${offsetWidth}px`
                // console.log(newValue);
                this.$refs.mmProgressInner.style.width=newValue;
                
            }
        },
        beforeDestroy () {
            this.unbindEvents()
        }
	}
</script>

<style lang="less">
    //进度条
    @bar_color: rgba(118, 255, 38, 0.85);
    @line_color: #fff;
    @dot_color: #fff;
	.mmProgress {
        position: relative;
        padding: 5px;
        user-select: none;
        cursor: pointer;
        overflow: hidden;
        .mmProgress-bar {
            height: 2px;
            width: 100%;
            background: @bar_color;
        }
        .mmProgress-outer {
            position: absolute;
            top: 50%;
            left: 5px;
            display: inline-block;
            width: 0;
            height: 2px;
            margin-top: -1px;
            background: rgba(255,255,255,.2);
        }
        .mmProgress-inner {
            position: absolute;
            top: 50%;
            left: 5px;
            display: inline-block;
            width: 0;
            height: 2px;
            margin-top: -1px;
            background: @line_color;
            .mmProgress-dot {
                position: absolute;
                top: 50%;
                right: -8px;
                width: 7px;
                height: 7px;
                border:3px solid #d44439;
                margin-left:2px;
                border-radius: 50%;
                background-color: @dot_color;
                transform: translateY(-50%);
            }
        }
	}
</style>