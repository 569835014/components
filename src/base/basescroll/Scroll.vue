<template>
  <div ref="wrapper" class="list-wrapper">
    <div class="scroll-content">
      <slot>
        <ul ref="list" class="list-content">
          <li @click="clickItem($event,item)" class="list-item" v-for="item in data">{{item}}</li>
        </ul>
      </slot>
      <slot name="pullup"
            :pullUpLoad="pullUpLoad"
            :isPullUpLoad="isPullUpLoad"
      >
        <div class="pullup-wrapper" v-if="pullUpLoad">
          <div class="before-trigger" v-if="!isPullUpLoad">
            <span>{{pullUpTxt}}</span>
          </div>
          <div class="after-trigger" v-else>
            <loading></loading>
          </div>
        </div>
      </slot>
    </div>
    <slot name="pulldown"
          :pullDownRefresh="pullDownRefresh"
          :pullDownStyle="pullDownStyle"
          :beforePullDown="beforePullDown"
          :pulling="pulling"
          :bubbleY="bubbleY"
    >
      <div ref="pulldown" class="pulldown-wrapper" :style="pullDownStyle" v-if="pullDownRefresh">
        <!--正在下拉还没触发下拉事件用beforePullDown来控制这个状态-->
        <div class="before-trigger" v-if="beforePullDown">
          <bubble :y="bubbleY"></bubble>
        </div>
        <!--正在异步获取数据-->
        <div class="after-trigger" v-else>
          <div v-if="pulling" class="loading">
            <loading></loading>
          </div>
          <!--数据渲染完成-->
          <div v-else><span>{{refreshTxt}}</span></div>
        </div>
      </div>
    </slot>
  </div>
</template>
<script>
  import BScroll from 'better-scroll'
  const COMPONENT_NAME = 'scroll'
  const DIRECTION_H = 'horizontal'
  const DIRECTION_V = 'vertical'
  import Loading from '../loading/loading.vue'
  import Bubble from '../bubble/bubble.vue'
  export default {
    name:COMPONENT_NAME,
    data(){
      return{
        beforePullDown: true,
        isRebounding: false,
        isPullingDown: false,
        pulling: false,
        isPullUpLoad: false,
        pullUpDirty: true,
        pullDownStyle: '',
        bubbleY: 0
      }
    },
    props:{
      data: {//数据
        type: Array,
        default: function () {
          return []
        }
      },
      probeType: {
        type: Number,
        default: 1
      },
      click: {//是否派发事件
        type: Boolean,
        default: true
      },
      listenScroll: {//是否监听滚动
        type: Boolean,
        default: false
      },
      listenBeforeScroll: {//是否监听滚动开始就前的事件
        type: Boolean,
        default: false
      },
      direction: {//滚动方向
        type: String,
        default: DIRECTION_V
      },
      scrollbar: {//是否有滚动条
        type: null,
        default: false
      },
      pullDownRefresh: {//是否开启上啦刷新
        type: null,
        default: false
      },
      pullUpLoad: {//是否开启下拉加载
        type: null,
        default: false
      },
      startY: {
        type: Number,
        default: 0
      },
      refreshDelay: {
        type: Number,
        default: 20
      },
      freeScroll: {//自由滚动
        type: Boolean,
        default: false
      }
    },
    created(){
      this.pullDownInitTop = -50
    },
    mounted(){
      this.$nextTick(function () {
        this.initScroll();
      })
    },
    computed:{
      pullUpTxt() {
        const moreTxt = this.pullUpLoad && this.pullUpLoad.txt && this.pullUpLoad.txt.more || "正在加载更多"

        const noMoreTxt = this.pullUpLoad && this.pullUpLoad.txt && this.pullUpLoad.txt.noMore || "已经是最后一页"

        return this.pullUpDirty ? moreTxt : noMoreTxt
      },
      refreshTxt() {
        return this.pullDownRefresh && this.pullDownRefresh.txt || '刷新完成'
      }
    },
    methods:{
      disable() {
        this.scroll && this.scroll.disable()
      },
      enable() {
        this.scroll && this.scroll.enable()
      },
      refresh() {
        this.scroll && this.scroll.refresh()
      },
      scrollTo(...arg) {
        this.scroll && this.scroll.scrollTo.apply(this.scroll, arg)
      },
      scrollToElement(...agr) {
        this.scroll && this.scroll.scrollToElement.apply(this.scroll,agr)
      },
      clickItem(e, item) {
        this.$emit('click', item,e)
      },
      destroy() {
        this.scroll.destroy()
      },
      //初始化betterscroll
      initScroll(){
        if (!this.$refs.wrapper) {
          throw new Error("没有主容器");
        }
        let options = {
          probeType: this.probeType,
          click: this.click,
          scrollY: this.freeScroll || this.direction === DIRECTION_V,
          scrollX: this.freeScroll || this.direction === DIRECTION_H,
          scrollbar: this.scrollbar,
          pullDownRefresh: this.pullDownRefresh,
          pullUpLoad: this.pullUpLoad,
          startY: this.startY,
          freeScroll: this.freeScroll
        }
        this.scroll=new BScroll(this.$refs.wrapper,options);
        //监听滚动事件
        if(this.listenScroll){
          this.scroll.on('scroll',(pos)=>{
            this.$emit('scroll',pos)
          })
        }
        //监听滚动开始前事件
        if(this.listenBeforeScroll){
          this.scroll.on('beforeScrollStart',()=>{
            this.$emit('beforeScrollStart')
          })
        }
        //开启上拉刷新，初始化上拉刷新函数
        if (this.pullDownRefresh) {
          this._initPullDownRefresh()
        }
        //开启下拉加载，初始化下拉加载
        if (this.pullUpLoad) {
          this._initPullUpLoad()
        }
      },
      _initPullDownRefresh(){
        this.scroll.on('pullingDown', () => {
          console.info(0)
          this.beforePullDown = false //下拉设置为false
          this.isPullingDown = true //现在为正在执行下拉函数的状态
          this.pulling = true //现在为正在执行下拉函数的状态
          this.$emit('pullingDown') //派发出事件
        })

        this.scroll.on('scroll', (pos) => {
          if (this.beforePullDown) {
            this.bubbleY = Math.max(0, pos.y + this.pullDownInitTop)
            this.pullDownStyle = `top:${Math.min(pos.y + this.pullDownInitTop, 10)}px`
          } else {
            this.bubbleY = 0
          }

          if (this.isRebounding) {
            this.pullDownStyle = `top:${10 - (this.pullDownRefresh.stop - pos.y)}px`
          }
        })
      },
      _initPullUpLoad() {
        this.scroll.on('pullingUp', () => {
          this.isPullUpLoad = true //下拉加载执行的事件
          this.$emit('pullingUp')
        })
      },
      _reboundPullDown() {//下拉回弹后执行的函数
        const {stopTime = 600} = this.pullDownRefresh
        return new Promise((resolve) => {
          setTimeout(() => {
            this.isRebounding = true //正在回弹
            this.scroll.finishPullDown() //调用finishPullDown函数执行回弹
            this.isPullingDown = false //正在下拉设置为false
            resolve()//下一步函数
          }, stopTime)
        })
      },
      async _afterPullDown() {//下拉刷新的最后一步
        await new Promise((resolve)=>{
          setTimeout(() => {
            this.pullDownStyle = `top:${this.pullDownInitTop}px`
            this.beforePullDown = true //设置为可以下拉
            this.isRebounding = false //正在回弹设置为false
            this.refresh()//刷新betterscroll
            resolve()
          }, this.scroll.options.bounceTime)
        })
        setTimeout(()=>{//执行完所有的下拉刷新事件触发的函数
          this.$emit('pullDownOver')
        },this.refreshDelay)
      },
      forceUpdate(dirty) {//上拉刷新下拉加载更新dom
        if (this.pullDownRefresh && this.isPullingDown) {
          this.pulling = false
          this._reboundPullDown().then(() => {
            this._afterPullDown()
          })
        } else if (this.pullUpLoad && this.isPullUpLoad) {
          this.isPullUpLoad = false
          this.scroll.finishPullUp()
          this.pullUpDirty = dirty
          this.refresh()
        } else {
          this.refresh()
        }
      },
    },
    watch:{
      data(){
        console.info(1)
        setTimeout(() => {//当数据发生变化延迟刷新scroll,dom更新也是需要时间的嘛，老黄的经验是20ms，但是我觉得这个最好设置高一点
          this.forceUpdate(true)
        }, this.refreshDelay)
      }
    },
    components:{
      Loading,
      Bubble
    }
  }
</script>
<style scoped lang="stylus">
  .list-wrapper
    position: absolute
    left: 0
    top: 0
    right: 0
    bottom: 0
    overflow: hidden
    background: #fff
    .list-content
      position: relative
      z-index: 10
      background: #fff
      .list-item
        height: 60px
        line-height: 60px
        font-size: 18px
        padding-left: 20px
        border-bottom: 1px solid #e5e5e5

  .pulldown-wrapper
    position: absolute
    width: 100%
    left: 0
    display: flex
    justify-content center
    align-items center
    transition: all
    .after-trigger
      margin-top: 10px

  .pullup-wrapper
    width: 100%
    display: flex
    justify-content center
    align-items center
    padding: 16px 0

</style>
