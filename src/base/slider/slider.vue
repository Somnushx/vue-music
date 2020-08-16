<template>
  <div class="slider" ref="slider">
    <div class="slider-group" ref="sliderGroup">
      <slot></slot>
    </div>
    <div class="dots">
      <span
        class="dot"
        v-for="(item, index) in dots"
        :key="index"
        :class="{ active: currentPageIndex === index }"
      ></span>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import BScroll from 'better-scroll'
import {addClass} from 'common/js/dom'

export default {
  name: 'slider',
  data() {
    return {
      dots: [],
      currentPageIndex: 0
    }
  },
  props: {
    loop: {
      type: Boolean,
      default: true // 是否是循环轮播
    },
    autoPlay: {
      type: Boolean,
      default: true // 是否是自动播放
    },
    interval: {
      type: Number,
      default: 4000 // 轮播间隔
    }
  },
  mounted() {
    // 设置定时器，使页面DOM渲染完成后初始化better-scroll，浏览器17ms刷新一次，一般设置成20ms
    setTimeout(() => {
      this._setSliderWidth() // 初始化better-scroll之前要计算元素的宽度
      this._initDots()
      this._initSlider() // 初始化better-scroll
      if (this.autoPlay) {
        this._play() // 初始化完成后进行自动轮播
      }
    }, 20)
    // 屏幕大小变化时，监听该变化，实现页面大小自适应
    window.addEventListener('resize', this.resize)
  },
  methods: {
    resize() {
      if (!this.slider) {
        return
      }
      this._setSliderWidth(true)
      this.slider.refresh() // 重新计算宽度后，刷新
    },
    // 计算
    _setSliderWidth(isResize) {
      this.children = this.$refs.sliderGroup.children
      let width = 0
      let sliderWidth = this.$refs.slider.clientWidth
      for (let i = 0; i < this.children.length; i++) {
        let child = this.children[i]
        addClass(child, 'slider-item') // 添加slider-item类名

        child.style.width = sliderWidth + 'px'
        width += sliderWidth
      }
      if (this.loop && !isResize) {
        width += 2 * sliderWidth // 如果slide是需要循环的话，slide会在左右两侧copy两个子元素从而保证循环切换
      }

      this.$refs.sliderGroup.style.width = width + 'px'
    },
    _initSlider() {
      this.slider = new BScroll(this.$refs.slider, {
        scrollX: true,
        scrollY: false,
        momentum: false, // 当快速滑动时是否开启滑动惯性
        snap: true,
        snapLoop: this.loop, // 是否可以无缝循环轮播
        snapThreshold: 0.3, // 用手指滑动时页面可切换的阈值，大于这个阈值可以滑动的下一页
        snapSpeed: 400 // 轮播图切换的动画时间
      })
      // 为滚动结束时添加事件
      this.slider.on('scrollEnd', () => {
        let pageIndex = this.slider.getCurrentPage().pageX
        if (this.loop) {
          pageIndex -= 1 // 因为设置自动播放之后，会自动在左右两侧copy两个子元素，所以在设置pageIndex时应减1
        }
        this.currentPageIndex = pageIndex
        if (this.autoPlay) {
          this._play() // 手动滚动后仍然进行自动播放
        }

        this.slider.on('beforeScrollStart', () => {
          if (this.autoPlay) {
            clearTimeout(this.timer) // 滚动开始前清除自动播放的定时器
          }
        })
      })
    },
    _initDots() {
      this.dots = new Array(this.children.length)
    },
    _play() {
      let pageIndex = this.currentPageIndex + 1
      if (this.loop) {
        pageIndex += 1 // 循环播放是会产生一个副本
      }
      this.timer = setTimeout(() => {
        this.slider.goToPage(pageIndex, 0, 400)
      }, this.interval)
    }
  },
  activated() {
    if (this.autoPlay) {
      this._play()
    }
  },
  deactivated() {
    clearTimeout(this.timer)
  },
  beforeDestroy() {
    window.removeEventListener('resize', this.resize)
    clearTimeout(this.timer) // 组件中设置定时器时，要在组件销毁前清除定时器
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable'

.slider
    min-height: 1px
    width: 100%
    .slider-group
      position: relative
      overflow: hidden
      white-space: nowrap
      .slider-item
        float: left
        box-sizing: border-box
        overflow: hidden
        text-align: center
        a
          display: block
          width: 100%
          overflow: hidden
          text-decoration: none
        img
          display: block
          width: 100%
    .dots
      position: absolute
      right: 0
      left: 0
      bottom: 12px
      text-align: center
      font-size: 0
      .dot
        display: inline-block
        margin: 0 4px
        width: 8px
        height: 8px
        border-radius: 50%
        background: $color-text-l
        &.active
          width: 20px
          border-radius: 5px
          background: $color-text-ll
</style>
