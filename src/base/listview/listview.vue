<template>
  <scroll
    class="listview"
    :data="data"
    ref="listview"
    :probe-type="probeType"
    :listenScroll="listenScroll"
    @scroll="scroll"
  >
    <ul>
      <li
        v-for="(item, index) in data"
        :key="index"
        class="list-group"
        ref="listGroup"
      >
        <h2 class="list-group-title">{{ item.title }}</h2>
        <ul>
          <li
            @click="selectItem(item)"
            v-for="(item, index) in item.items"
            :key="index"
            class="list-group-item"
          >
            <img class="avatar" v-lazy="item.avatar" />
            <span class="name">{{ item.name }}</span>
          </li>
        </ul>
      </li>
    </ul>
    <div
      class="title-list"
      @touchstart.stop.prevent="onShortcutTouchStart"
      @touchmove.stop.prevent="onShortcutTouchMove"
    >
      <ul>
        <li
          v-for="(item, index) in data"
          :key="index"
          class="title-list-item"
          :data-index="index"
          :class="{ current: currentIndex === index }"
        >
          {{ item.title == "热门" ? "热" : item.title }}
        </li>
      </ul>
    </div>
    <div class="list-fixed" v-show="fixedTitle" ref="fixed">
      <h2 class="fixed-title">{{ fixedTitle }}</h2>
    </div>
    <div v-show="!data.length" class="loading-container">
      <loading></loading>
    </div>
  </scroll>
</template>

<script type="text/ecmascript-6">
import Scroll from 'base/scroll/scroll'
import {getData} from 'common/js/dom'
import Loading from '../loading/loading'

const ANCHOR_HEIGHT = 18
const TITLE_HEIGHT = 30
export default {
  data() {
    return {
      scrollY: -1,
      currentIndex: 0,
      diff: -1
    }
  },
  props: {
    data: {
      type: Array,
      default: () => {}
    }
  },
  created() {
    this.touch = {}
    this.listenScroll = true // 是否监听滚动事件
    this.listHeight = []
    this.probeType = 3 // 除了实时派发scroll事件，在滑动情况下仍然能实时派发scroll事件
  },
  computed: {
    fixedTitle() {
      if (this.scrollY > 0) {
        return
      }
      return this.data[this.currentIndex] ? this.data[this.currentIndex].title : ''
    }
  },
  components: {
    Scroll,
    Loading
  },
  methods: {
    onShortcutTouchStart(e) {
      // 在DOM中为字母列表添加data-index属性，通过getData()获取当前字母列表data-index属性对应的index值,用getAttribute()实现
      // e.target代表当前点击元素，点击右侧对应元素，左侧滚动到对应元素
      let anchorIndex = getData(e.target, 'index') // 获取当前点击元素的索引，这个索引是字符串
      let firstTouch = e.touches[0] // 记录刚开始触碰的位置
      this.touch.y1 = firstTouch.pageY // 记录y轴方向的位置
      this.touch.anchorIndex = anchorIndex // 设置点击的索引，共享在两个函数之间
      // 传入当前元素的索引值，滚动到对应元素的位置
      // 通过给歌手列表添加ref实现，每个index对应一个listGroup
      this._scrollTo(anchorIndex)
    },
    // 给右侧字母栏，添加监听事件onShortcutTouchMove事件，实现在右侧滑动，左侧依然跟着联动
    // 滑动字母列表时触发，点击字母列表中的字母时不触发
    onShortcutTouchMove(e) {
      let firstTouch = e.touches[0] // 记录滑动后松手的位置
      this.touch.y2 = firstTouch.pageY // 手指停止移动的位置
      let delta = (this.touch.y2 - this.touch.y1) / ANCHOR_HEIGHT | 0 // 计算偏移锚点
      let anchorIndex = parseInt(this.touch.anchorIndex) + delta // 将获得的字符串索引，转化成数字 开始锚点加上偏移锚点
      this._scrollTo(anchorIndex) // 滚动到对应元素的位置
    },
    // 滚动时触发，接收scroll组件中传入的pos值
    scroll(pos) {
      this.scrollY = pos.y
    },
    _scrollTo(index) {
      // 滚到字母列表顶部和底部，即超过字母列表的范围时，不执行任何操作
      if (!index && index !== 0) {
        return
      }
      if (index < 0) {
        index = 0
      } else if (index > this.listHeight.length - 2) {
        index = this.listHeight.length - 2
      }
      this.scrollY = -this.listHeight[index]
      this.$refs.listview.scrollToElement(this.$refs.listGroup[index], 0)
    },
    // 计算歌手列表中每个列表组合的高度
    _calculateHeight() {
      this.listHeight = []
      let list = this.$refs.listGroup
      let height = 0
      this.listHeight.push(height)
      for (let i = 0; i < list.length; i++) {
        let item = list[i]
        height += item.clientHeight
        this.listHeight.push(height)
      }
    },
    selectItem(item) {
      this.$emit('select', item)
    }
  },
  watch: {
    data() {
      // 页面刷新时，重新计算歌手列表的高度
      setTimeout(() => {
        this._calculateHeight()
      }, 20)
    },
    // scrollY变化的时候，根据列表高度判断当前位于哪个区间，具体用currentIndex作为判断依据
    scrollY(newY) {
      let listHeight = this.listHeight
      // 当滚动到顶部时
      if (newY > 0) {
        this.currentIndex = 0
        return
      }
      // 当滚动到中部时
      for (let i = 0; i < listHeight.length - 1; i++) {
        let height1 = listHeight[i]
        let height2 = listHeight[i + 1]
        // 从下往上滑动时，newY为负数；从上往下滑动时，newY逐渐变为正数
        if ((-newY) >= height1 && (-newY) < height2) {
          this.currentIndex = i
          this.diff = height2 + newY
          // console.log('高度', height2, newY, this.diff)
          return
        }
      }
      // 当滚动到底部,且-newY大于最后一个元素的上限时
      this.currentIndex = this.listHeight.length - 2
    },
    // 左侧列表顶部索引字母更新
    diff(newVal) {
      let fixedTop = (newVal > 0 && newVal < TITLE_HEIGHT) ? newVal - TITLE_HEIGHT : 0
      // console.log('固定高度', this.fixedTop, fixedTop)
      if (this.fixedTop === fixedTop) {
        return
      }
      this.fixedTop = fixedTop
      this.$refs.fixed.style.transform = `translate3d(0,${fixedTop}px,0)`
    }
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable'

.listview
  position: relative
  width: 100%
  height: 100%
  overflow: hidden
  background: $color-background
  .list-group
    padding-bottom: 30px
    .list-group-title
      height: 30px
      line-height: 30px
      padding-left: 20px
      font-size: $font-size-small
      color: $color-text-l
      background: $color-highlight-background
    .list-group-item
      display: flex
      align-items: center
      padding: 20px 0 0 30px
      .avatar
        width: 50px
        height: 50px
        border-radius: 50%
      .name
        margin-left: 20px
        color: $color-text-l
        font-size: $font-size-medium
  .title-list
    position: absolute
    z-index: 30
    right: 0
    top: 50%
    transform: translateY(-50%)
    width: 20px
    padding: 20px 0
    border-radius: 10px
    text-align: center
    background: $color-background-d
    font-family: Helvetica
    .title-list-item
      padding: 3px
      line-height: 1
      color: $color-text-l
      font-size: $font-size-small
      &.current
        color: $color-theme
  .list-fixed
    position: absolute
    top: 0
    left: 0
    width: 100%
    .fixed-title
      height: 30px
      line-height: 30px
      padding-left: 20px
      font-size: $font-size-small
      color: $color-text-l
      background: $color-highlight-background
  .loading-container
    position: absolute
    width: 100%
    top: 50%
    transform: translateY(-50%)
</style>
