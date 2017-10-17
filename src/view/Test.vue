<template>

  <div>
    <scroll ref="scroll"
            class="warp-box"
            :data="list"
            :listenScroll="true"
            @scroll="move"
            @pullingUp="pullUp"
            :pullDownRefresh="{
            stop:40,
            threshold:90
          }"
            :scrollbar="true"
            :pullUpLoad="true"
            @pullingDown="downFn"
            :listenBeforeScroll="true"
            @beforeScrollStart="moveStart"
    >
      <ul>
        <li v-for="(item,index) in list">
          <div>{{`I MA item${index}`}}</div>
        </li>
      </ul>
    </scroll>

  </div>
</template>
<script>
  import Scroll from '../base/basescroll/Scroll.vue'

  export default {
    data() {
      return {
        li: 20,
        page: 4,
        list: []
      }
    },
    created() {
      this.createdArr(20);
      this.activeIndex = 1
    },
    methods: {
      move(pos) {

      },
      downFn() {
        this.activeIndex=1
        setTimeout(() => {
          this.createdArr(20)
        }, 1500)

      },
      pullUp() {
        this.activeIndex++
        if (this.activeIndex <= 4) {
          setTimeout(() => {
            this.createdArr(this.activeIndex * 20)
          }, 1500)
        } else {
          this.$refs.scroll.forceUpdate();
        }
      },
      moveStart() {

      },
      createdArr(num) {
        let arr = [];
        for (let i = 0; i < num; i++) {
          arr.push(i)
        }
        this.list = arr
      }
    },
    components: {
      Scroll
    }
  }
</script>
<style scoped lang="stylus">
  * {
    margin 0px
    padding 0px

  }

  .warp-box
    width 100%
    height 100%
    position fixed
    top 0
    left 0
    overflow hidden
    ul
      top: 0
      padding-left 0
    li:first-child
      border-top: 1px solid #e5e5e5;
    li
      list-style none
      height: 60px;
      line-height: 60px;
      font-size: 18px;
      border-bottom: 1px solid #e5e5e5;
      div
        padding-left: 20px;
</style>
