<template>
  <div class="progress-bar" ref="progressBar">
    <div class="bar-inner">
      <div class="progress" ref="progress"></div>
      <div
        class="progress-btn-wrapper"
        ref="progressBtn"
        @touchstart.prevent="progressTouchStart"
        @touchmove.prevent="progressTouchMove"
        @touchend="progressTouchEnd"
      >
        <div class="progress-btn"></div>
      </div>
    </div>
  </div>
</template>

<script type="text/ecmascript-6">
import {prefixStyle} from 'common/js/dom'
const progressBtnWidth = 16
const transform = prefixStyle('transform')

export default {
  data() {
    return {
      touch: {}
    }
  },
  props: {
    percent: {
      type: Number,
      default: 0
    }
  },
  // created() {
  //   this.touch = {}
  // },
  watch: {
    percent(newpercent) {
      if (newpercent >= 0 && !this.touch.initiated) {
        let barWidth = this.$refs.progressBar.clientWidth - progressBtnWidth
        let offsetWidth = newpercent * barWidth
        this._offset(offsetWidth)
      }
    }
  },
  methods: {
    progressTouchStart(e) {
      this.touch.initiated = true
      this.touch.startX = e.touches[0].pageX
      this.touch.left = this.$refs.progress.clientWidth
    },
    progressTouchMove(e) {
      if (!this.touch.initiated) {
        return
      }
      const delta = e.touches[0].pageX - this.touch.startX
      const offsetWidth = Math.min(this.$refs.progressBar.clientWidth - progressBtnWidth, Math.max(0, this.touch.left + delta))
      this._offset(offsetWidth)
    },
    progressTouchEnd() {
      this.touch.initiated = false
    },
    _offset(offsetWidth) {
      this.$refs.progress.style.width = `${offsetWidth}px`
      this.$refs.progressBtn.style[transform] = `translate3d(${offsetWidth}px, 0, 0)`
    }
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import "~common/stylus/variable"

.progress-bar
  height: 30px
  .bar-inner
    position: relative
    height: 4px
    top: 13px
    background: rgba(0, 0, 0, 0.3)
    .progress
      position: absolute
      height: 100%
      background: $color-theme
    .progress-btn-wrapper
      position: absolute
      width: 30px
      height: 30px
      left: -8px
      top: -13px
      .progress-btn
        position: relative
        left: 7px
        top: 7px
        width: 16px
        height: 16px
        box-sizing: border-box
        border-radius: 50%
        border: 3px solid $color-text
        background: $color-theme
</style>
