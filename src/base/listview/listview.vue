<template>
  <scroll class="listview" :data="data" ref="listview" :probeType="probeType" :listenScroll="listenScroll" @scroll='scroll'>
    <ul>
      <li v-for="(group,index) in data" class="list-group" :key="index" ref="listGroup">
        <h2 class="list-group-title">{{group.title}}</h2>
        <ul>
          <li @click="selectItem(item)" v-for="(item,index) in group.items" class="list-group-item" :key="index">
            <img class="avatar" v-lazy="item.avatar" alt="">
            <span class="name">{{item.name}}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div class="list-shortcut" @touchstart="onShortcutTouchStart" @touchmove.stop.prevent="onShortcutTouchMove">
      <ul>
        <li v-for="(item,index) in shortcutList" :class="{'current':currentIndex == index}" :key="index" class="item" :data-index="index">{{item}}</li>
      </ul>
    </div>
    <div class="list-fixed" v-show="fixedTitle" ref="fixed">
      <h1 class="fixed-title">{{fixedTitle}}</h1>


    </div>
    <div v-show="!data.length" class="loading-container">
      <loading></loading>>

    </div>
  </scroll>

</template>

<script>
  import Scroll from 'base/scroll/scroll'
  import Loading from 'base/loading/loading'
  import { getData } from 'common/js/dom'

  const TITLE_HEIGHT = 30
  const ANCHOR_HEIGHT = 18
  // 1 右边：字母设置触摸开始和触摸移动事件，获取当前经过的index，并设置高亮，获取整体页面移动的scrollY
  // 2 左边：滑动左边页面整体内容，根据所得到的的每一个部分的开始高度和结束高度，以及实时监听到的scrollY,计算出右边字母哪一个高亮的下标
  // 3 顶部：滑动中，实时计算每一个区域块的diff（height2 + scrollY）,如果diff小于TITLE_HEIGHT，往上translate，如果大于的话，设置translate为0，出现动画效果
  // 上述三个步骤，实现完整的联动

  export default {

    created() {
      // 不观测，写在这里
      this.touch = {}
      this.listenScroll = true
      this.listHeight= []
      this.probeType = 3

    },
    // 写在data里主要是为了和dom做一个监听，vue会添加getter,setter
    data() {
      return {
      scrollY:-1,
      currentIndex:0,
      diff:-1


      }

    },
    props: {
      data: {
        type: Array,
        default: []
      }
    },
    computed: {
      shortcutList() {
        return this.data.map((group) => {
          return group.title.substr(0, 1)
        })
      },
      fixedTitle() {
        if (this.scrollY > 0) {
          return
        }
        // console.log(this.data[this.currentIndex])
        return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
      }
    },

    methods: {
      selectItem(item) {
        this.$emit('select',item)
      },
      onShortcutTouchStart(e) {
        // 手指点击，获取右边每个字母的下标
        let anchorIndex = getData(e.target, 'index')
        let firstTouch = e.touches[0]
        this.touch.y1 = firstTouch.pageY //记录开始移动的坐标
        this.touch.anchorIndex = anchorIndex
        this.$refs.listview.scrollToElement(this.$refs.listGroup[anchorIndex], 0)
        this._scrollTo(anchorIndex)
      },
      onShortcutTouchMove(e) {
        let firstTouch = e.touches[0]
        this.touch.y2 = firstTouch.pageY
        // 移动了几个ANCHOR_HEIGHT，下标就加上几
        let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0
        let anchorIndex = parseInt(this.touch.anchorIndex) + delta
        this._scrollTo(anchorIndex)

      },
      scroll(pos) {
        // console.log(pos)
        this.scrollY = pos.y

      },
      refresh() {
        this.$refs.listview.refresh()
      },
      _scrollTo(index) {
        if (!index && index !=0) {
          return  
        }
        if (index < 0) {
          index = 0
        }else if (index > this.listHeight.length - 2) {
          index = this.listHeight.length - 2
        }
        this.scrollY = -this.listHeight[index]
        // console.log(this.scrollY)
        this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)

      },
      _calculateHeight() {
        this.listHeight = []
        const list = this.$refs.listGroup
        let height = 0
        this.listHeight.push(height)
        for (var i = 0; i<list.length; i++) {
          let item = list[i]
          height += item.clientHeight
          this.listHeight.push(height)
        }
        console.log(this.listHeight)

      }

    },
    watch: {
      data(){
        setTimeout(() => {
          this._calculateHeight()
        },20)
      },
      scrollY(newY) {
        console.log(newY,'newY')
        const listHeight = this.listHeight
        // 当滚动到顶部
        if (newY > 0) {
          this.currentIndex = 0
          return
        }
        // 在中间部分滚动
        for (var i =0; i< listHeight.length-1; i++) {
          // console.log(i)
          let height1 = listHeight[i]
          let height2 = listHeight[i+1]
          if (-newY >= height1 && -newY < height2) {
            // console.log(height2)
            this.currentIndex = i
            this.diff = height2 + newY
            return 
          }
          // console.log(this.currentIndex)
        }
        // 当滚动到底部，且n-ewY要大于最后一个元素的上线

        this.currentIndex = listHeight.length - 2
      },
      diff(newVal) {
        console.log(newVal,'newVal')
        let fixedTop = (newVal > 0 && newVal < TITLE_HEIGHT) ? newVal - TITLE_HEIGHT : 0
        console.log(fixedTop,'fixedTop')
        if (this.fixedTop == fixedTop) {
          return
        }
        this.fixedTop = fixedTop 
      
        this.$refs.fixed.style.transform = `translate3d(0,${fixedTop }px,0)`
      }

    },
    components: {
      Scroll,
      Loading
    }
  }

</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
  @import '~common/stylus/variable';

  .listview {
    position: relative;
    width: 100%;
    height: 100%;
    overflow: hidden;
    background: $color-background;

    .list-group {
      padding-bottom: 30px;

      .list-group-title {
        height: 30px;
        line-height: 30px;
        padding-left: 20px;
        font-size: $font-size-small;
        color: $color-text-l;
        background: $color-highlight-background;
      }

      .list-group-item {
        display: flex;
        align-items: center;
        padding: 20px 0 0 30px;

        .avatar {
          width: 50px;
          height: 50px;
          border-radius: 50%;
        }

        .name {
          margin-left: 20px;
          color: $color-text-l;
          font-size: $font-size-medium;
        }
      }
    }

    .list-shortcut {
      position: absolute;
      z-index: 30;
      right: 0;
      top: 50%;
      transform: translateY(-50%);
      width: 20px;
      padding: 20px 0;
      border-radius: 10px;
      text-align: center;
      background: $color-background-d;
      font-family: Helvetica;

      .item {
        padding: 3px;
        line-height: 1;
        color: $color-text-l;
        font-size: $font-size-small;

        &.current {
          color: $color-theme;
        }
      }
    }

    .list-fixed {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;

      .fixed-title {
        height: 30px;
        line-height: 30px;
        padding-left: 20px;
        font-size: $font-size-small;
        color: $color-text-l;
        background: $color-highlight-background;
      }
    }

    .loading-container {
      position: absolute;
      width: 100%;
      top: 50%;
      transform: translateY(-50%);
    }
  }
</style>