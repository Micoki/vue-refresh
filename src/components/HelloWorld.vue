<template>
  <div class="hello">
      <pull-refresh v-model="refresh"
      :pullColors="'red'"
      :scrollHide="true"
      :finished="finishends"
      finishedText="到底了，没有啦！"
      @refresh="refershs"
      @load="loads">
          <p v-for="a in refreshs">{{a}}</p>
          <p v-for="a in 30">加载！加载！加载！</p> 
          <p v-for="a in dates">{{a}}</p>
      </pull-refresh>
  </div>
</template>

<script> 
 // 防抖   配合 上拉加载数据
function debounce(fn, delay) {    
    var delay = delay || 300;
    var timer;
    return function () {
        var th = this;            //设置this指向
        var args = arguments;     //设置伪数组对象指向
        if (timer) {
            clearTimeout(timer);
        }
        timer = setTimeout(function () {
            timer = null;
            fn.apply(th, args);
        }, delay);      //回调时间
    }; 
}
//节流
function throttle(fn, interval) {
    var last;
    var timer;
    var interval = interval || 200;
    return function () {
        var th = this;
        var args = arguments;
        var now = +new Date();
        if (last && now - last < interval) {
            clearTimeout(timer);
            timer = setTimeout(function () {
                last = now;
                fn.apply(th, args);
            }, interval);
        } else {
            last = now;
            fn.apply(th, args);
        }
    }
} 
export default {
  name: 'HelloWorld',
  data () {
    return {
       refresh:true,
       finishends:true,
       refreshs:[Math.random()*1134,Math.random()*1224,Math.random()*3134,Math.random()*1100],
       dates:['下拉加载'],
       num:0
    }
  },
  methods: {
      refershs(){
          setTimeout(()=>{
            this.refreshs = [Math.random()*1134,Math.random()*1224,Math.random()*3134,Math.random()*1100]
            this.refresh = false
          },3000)
      },
      // 建议配合防抖进行上拉加载数据
      loads:debounce(function(){
        this.num++
        if(this.num > 4){
          this.finishends = false
        }
        console.log('上拉加载成功！')
        this.dates.push('在上啦就加载','继续继续继续','上拉下拉','在上啦就加载','继续继续继续','上拉下拉',)
      },1000) 
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
 .hello{
   width: 100%; height: 100%;
 }
</style>
