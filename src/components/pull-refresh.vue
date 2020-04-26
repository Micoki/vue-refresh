<template>
    <div class="pull-refresh-box-content" :style="pullRefresh" :class="{scrollclose:scrollHide}"> 
      <!-- 加载动画区域 -->
      <div class="pull-refresh-box"
      :style="style"> 
          <div class="pull-arrow-circle">
              <div class="box-load" v-if="moveState == 2">
                <div class="left-load"> 
                  <div class="left-circle" :style="{backgroundColor:pullColors}"></div>
                </div>
                <div class="right-load"> 
                  <div class="right-circle" :style="{backgroundColor:pullColors}"></div>
                </div>
                <div class="among-load"></div>
              </div>
              <div class="pull-circle-line" :style="circleLine" v-else>
                <div class="pull-arrow">
                    <div class="arrows" :class="{opacity:rotateArrow}" :style="{ borderBottomColor:pullColors}"></div>
                </div>
              </div>   
          </div>
      </div>
      <!-- 顶部缓冲区域-->
      <div class="pull-buffer" :style="{height:topButters + 'px'}"></div> 
      <!-- 低部缓冲区域 -->
      <div class="pull-buffer-bottom" :style="{height:bottomButters + 'px'}"></div>
      <!-- 内容区域 -->
      <div
        class="pull-refresh" 
        @touchstart="touchStart"
        @touchmove="touchMove"
        @touchend="touchEnd" 
        @scroll="scrolls">
        <div class="pull-content-box"> 
            <!-- slot 插槽 组件使用内容区域 -->
          <slot/>   
          <!-- 上拉加载数据动画 -->
          <div v-show="finished" class="pull-add-load"> 
            <div class="box-load">
              <div class="left-load"> 
                <div class="left-circle" :style="{backgroundColor:pullColors}"></div>
              </div>
              <div class="right-load"> 
                <div class="right-circle" :style="{backgroundColor:pullColors}"></div>
              </div>
              <div class="among-load"></div>
            </div>
          </div>
          <!-- 底部上拉全部数据加载完提示 -->
          <div v-show="!finished && finishend" class="pull-add-load">
              <span class="finished-bottom">{{finishedText}}</span>
          </div>
        </div> 
      </div>
    </div>
</template>
 
<script>  
/**
 * api
 * @props 参数
 * v-model          false关闭下拉动画
 * pullColors       动画颜色
 * pullWidth        设置宽    
 * pullHeight       设置高
 * offset           滚动条与底部的距离小于多少触发上拉加载
 * scrollHide       是否隐藏滚动条，默认为 false 显示滚动条
 * finished         是否开启上拉加载，默认关闭
 * finishedText     设置上拉加载完后的提示
 * 
 * @refresh         下拉刷新回调函数
 * @load            上拉加载回调函数
 */
export default {
  name: 'pull-refresh',
  props: {                     //api props参数
    pullColors: {              //动画颜色
      type:String,
      default:'#1A73E8'       //默认为蓝色
    },
    pullWidth: {               //宽度
      type:Number,           //默认为100% 
    },
    pullHeight: {           //高度
      type:Number,         //默认为100% 
    }, 
    offset: {             //滚动条与底部的距离小于多少触发上拉加载，默认值为30
      type:Number,
      default:50        //默认为50             
    },
    scrollHide:{              //true 为隐藏滚动条,默认为 false 显示滚动条
      type:Boolean,
      default:false
    },
    finished:{                //是否开启上拉加载，默认关闭
      type:Boolean, 
      default:false
    },
    finishedText:{                    //上拉加载完后的提示
      type:String,             
      default:'到底了，没数据了！'    //默认值
    },
    value: {                   //v-model双向数据绑定
      type:Boolean,            //设置只能为布尔值，false下拉刷新结束
      default:true,           //默认true
      required:true,            //必填
    }
  }, 
  data () {
    return {  
        startY: '',             //保存第一次点击屏幕时的Y坐标
        moveDistance: -35,      //设置动画距离顶部的位置
        moveState: 0,           //开始滑动到结束后状态的变化 0:下拉即可刷新 1:释放即可刷新 2:加载中
        scrollStaus:true,       //添加滚动条到顶部时触发下拉刷新限制，增加用户体验感
        duration: 0,            //动画持续时间(毫秒)，0就是没有动画
        scrollTop:0,            //记录滚动条是否在顶部
        scales:1,               //记录刷新完成后缩小隐藏
        rotateCircle:0,         //记录线条圆圈旋转度数
        rotateArrow:false,      //记录下拉显示线条圆圈箭头
        opacity:0.3,            //记录可刷新颜色透明度
        scrollTops:0,           //记录是否有滚动条
        detimes:1000,           //设置防抖时间
        timers:null,            //设置防抖触发
        topButters:0,           //设置顶部缓冲阴影大小 
        bottomButters:0,        //设置地部缓冲阴影大小
        finishend:false,        //设置底部数据完全加载完提示显示
    }
  },
  computed: {
    pullRefresh(){
      return {
        width:this.pullWidth?this.pullWidth+'px':'100%',
        height:this.pullHeight?this.pullHeight+'px':'100%'
      }
    },
    style () {
      return {
        transition: `${this.duration}ms`,
        transform: `translate3d(0,${this.moveDistance}px, 0) scale(${this.scales})`, 
      }
    },
    circleLine(){
      return {
        borderColor:this.pullColors,
        transition: `${this.duration}ms`,                           //改变过渡时间
        transform: `rotate(${100 + this.rotateCircle}deg)`,         //改变旋转角度
        opacity:this.opacity,                                       //改变颜色透明度
      }
    }, 
  },
  methods: {   
    scrolls (e) {  
      this.scrollTops = e.target.scrollTop
      let offsets = e.target.scrollHeight - (e.target.scrollTop + e.target.clientHeight)    //计算滚动条到底部的距离
      let loads = this.offset                      //用户设置滚动条到底部距离多少触发上拉加载方法，默认值为30
      //手写小防抖，防止上拉多次触发上拉加载方法 
      //防抖与节流详情可以自寻百度 
      if(this.finished && this.moveState != 2){                  //判断是否开启上拉加载
        if (this.timers != null) {             
            clearTimeout(this.timers);
        }  
        this.timers = setTimeout(()=>{
            this.timers = null;
            if(offsets <= loads){             //当滚动条小于等于指定距离时触发回调函数
                this.$emit('load') 
            } 
        },500);         //设置200ms触发回调函数
      }
      //可滚动区域大于可视区域才显示上拉加载所有数据加载完提示
      if(e.target.scrollHeight > e.target.clientHeight){
        this.finishend = true
      }
      //阻止滚动条过快从下到上触发下拉刷新
      if(e.target.scrollTop == 0){  
        this.topButters = 80                 //显示缓冲区域 
        setTimeout(()=>{                  //设置滚动条到顶部时500ms后才可以下拉刷新
          this.topButters = 0 
          this.scrollStaus = true
        },700)
      }else{  
        this.scrollStaus = false
      }
      //显示底部缓冲区域
      if(offsets < 1){
        this.bottomButters = 70
        setTimeout(()=>{                  
          this.bottomButters = 0  
        },500)
      }
    },   
    touchStart (e) {
        if(!this.scrollStaus)     return;         //阻止滚动条到顶部时立即触发下拉刷新
        if(this.moveState == 2) return;           //判断是否在更新中 
        this.duration = 0                         // 关闭动画
        this.moveDistance = -35                   // 滑动距离归0 
        this.scales = 1                           // 初始化大小
        this.rotateArrow = false                  // 箭头初始化
        this.startY = e.targetTouches[0].clientY  // 获得开始Y坐标
    },
    touchMove (e) {                             //这里是整个下拉刷新的核心  
        if(!this.scrollStaus)     return;       //阻止滚动条到顶部时立即触发下拉刷新
        if(this.moveState == 2) return;         //判断是否在更新中 
        //判断滚动条是否在顶部，如果不在，则下拉刷新就不能启用。 
        if (this.scrollTops != 0) return;  
        let move = e.targetTouches[0].clientY - this.startY
        //判断手指滑动的距离，只有为正数才代表用户下拉了。
        if (move < 200) {     //限制下拉距离
            //阻止默认事件，在微信浏览器中尤为有用，至于为什么，你去试就知道了。
            // e.preventDefault() 
            //增加滑动阻力的感觉
            this.moveDistance = Math.pow(move, 0.9) - 35      //move的0.9次方
            this.rotateCircle = Math.pow(move, 1.2)           //下拉旋转度数 
            //显示线条圆圈箭头
            if(this.moveDistance > 15) { this.rotateArrow = true}   
            else {this.rotateArrow = false }

            if (this.moveDistance > 40) {
                //如果滑动距离大于50 那我就告诉你，释放即可刷新
                this.opacity = 1
                if (this.moveState === 1) return
                this.moveState = 1
            } else {
                //否则 恢复原样 
                this.opacity = 0.3
                if (this.moveState === 0) return
                this.moveState = 0
            } 
        }
    },
    touchEnd (e) { 
        if(this.moveState == 2) return;         //判断是否在更新中
        // 只要手指拿开，我都需要加上结束时的动画，这里为300ms
        this.duration = 300
        if (this.moveDistance > 50) {
            //这里逻辑跟touchMove一样，但是需要真的加载数据了，那moveState变为2 所以加载动画在这出现
            this.moveState = 2
            //因为还没加载完，我得让加载动画显示着，所以这里移动距离为25
            this.moveDistance = 25 

              //这里就是成功后的回调了
            this.$emit('refresh') 
        } else {
            //否则 给我老老实实恢复原样
            this.moveDistance = -35
        }
    },

  },
  watch: { 
    moveState (state) {
      //监听moveState的状态，0意味着开始也意味着结束，这里是结束，并且只有动画生效我们才能 moveDistance 设为0 
      //手指离开了屏幕，动画生效
      if (state === 0 && this.duration === 300) {
          this.moveDistance = -35
      }
    },
    value: {               //监听v-model值为false则关闭下拉刷新
      handler (val) {  
        if(!val){
          this.scales = 0 
          setTimeout(()=>{ 
            this.moveState = 0  
          },400)
          this.$emit('input',true)      //改变父组件v-model值为true,可以重复下拉刷新
        }
      },
      deep:true       //深度监听
    }, 
  }
}
</script> 
<style scoped>
.pull-refresh-box-content{
  position: relative; 
  overflow: hidden;
  background-color: #fff;
}
.pull-refresh { 
    position: relative;  
    overflow-y: auto;
    width: 100%; height: 100%; 
}
/* 隐藏滚动条 */

.scrollclose .pull-refresh::-webkit-scrollbar{
  display:  none;
}
.pull-refresh-box{
    position: absolute;
    display: flex; 
    justify-content: center;
    width: 100%;
    font-size: 14px;
    color: rgba(69,90,100,.6);
    text-align: center;  
    z-index: 500;
}
.pull-buffer{
    position: absolute; 
    top: -40px;
    width: 100%;
    border-radius:50%;  
    transition: all 500ms; 
    background-color: rgb(48, 47, 47,.1);
}
.pull-buffer-bottom{
    position: absolute; 
    bottom: -35px;
    width: 100%;
    z-index: 555;
    border-radius:50%;  
    transition: all 500ms; 
    background-color: rgb(48, 47, 47,.1);
}
.pull-content-box{
    width: 100%;  
}
.pull-arrow-circle{
  position: relative;
  width: 30px; height: 30px;
  border-radius: 50%;
  box-shadow: 0 0 2px #000;
  background-color: #fff;
  display: flex; 
  justify-content: center;
  align-items: center;
}
.pull-circle-line{
  position: relative;
  width: 15px; height: 15px;
  border-radius: 50%;
  background-color: #fff;
  box-sizing: border-box;
  border-width: 2px; 
  border-style: solid; 
  display: flex;
  align-items: center;
  justify-content: center;  
  /* transform: rotate(100deg); */
} 
.pull-arrow{
  position: absolute;
  left: -3px; 
  transform: rotate(120deg); 
  background-color: #fff;
  /* border-radius: 50%; */
  width: 4px; height: 6px;
}
.arrows{
  width:0; 
  height:0; 
  position: absolute;
  right: -4px; bottom: 4px;
  transform: rotate(-30deg);   
  border-bottom-width: 6px ;
  border-bottom-style:  solid;   
  border-right: 6px solid transparent;
  transition: .5s;
  opacity: 0;
}
.opacity{
  opacity: 1;
} 
 

.box-load{
  position: relative; 
  width: 18px; height: 18px;
  border-radius: 50%;
  overflow: hidden; 
  animation: boxs 3s infinite linear forwards;
}
.among-load{
  top: 2px;
  left: 2px;
  width: 14px;
  height: 14px;
  border-radius: 50%;
  position: absolute;
  background:#fff;
  z-index: 5; 
  overflow: hidden; 
}
.left-load{
  position: absolute;
  width: 50%;
  height: 18px;  
  background: #fff;
  overflow: hidden;
}
.right-load{
  left: 9px;
  position: absolute;
  width: 50%;
  height: 18px;  
  background: #fff;
  overflow: hidden;
}
.left-circle {
  position: absolute; 
  width: 9px;
  height: 18px;
  border-radius:18px 0px 0px 18px;   
  transform: rotate(-180deg);
  transform-origin: 9px 9px;  
  animation: pr1A 3s infinite linear forwards;
} 
.right-circle {
  position: absolute; 
  width: 9px;
  height: 18px;
  border-radius: 0px 18px 18px 0px;  
  transform: rotate(-180deg);
  transform-origin: 0px 9px;   
  animation: pr2A 3s infinite linear forwards;
} 
@keyframes pr1A {
  0% { 
    transform: rotate(-180deg);
  }
  25% { 
    transform: rotate(-180deg);
  }
  50% { 
    transform: rotate(0deg);
  }
  75%{
    transform: rotate(0deg);
  }
  100%{
    transform: rotate(180deg);
  }
}
@keyframes pr2A {
  0% { 
    transform: rotate(-180deg);
  }
  25% { 
    transform: rotate(0deg);
  }
  50% { 
    transform: rotate(0deg);
  }
  75%{
    transform: rotate(180deg);
  }
  100%{
    transform: rotate(180deg);
  }
}
@keyframes boxs {
  0% { transform: rotate(0deg) }
  100% { transform: rotate(360deg) }
}

.pull-add-load{ 
    width: 100%;
    padding: 15px 0; 
    display: flex;
    justify-content: center;
}
.finished-bottom{
    font-size: 12px;
    color: #777;
}
</style>