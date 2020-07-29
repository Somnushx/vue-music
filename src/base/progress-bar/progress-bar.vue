<template>
  <div class='progress-bar' ref="progressBar">
    <div class="bar-inner">
      <div class="progress" ref="progress"></div>
      <div class="progress-btn-wrapper" ref="progressBtn">
        <div class="progress-btn"></div>
      </div>
    </div>
  </div>
</template>

<script type='text/ecmascript-6'>
import {prefixStyle} from 'common/js/dom'
const progressBtnWidth = 16
const transform = prefixStyle('transform')

export default {
  props: {
    percent: {
      type: Number,
      default: 0
    }
  },
  watch: {
    percent(newpercent) {
      console.log('newpercent', newpercent)
      let barWidth = this.$refs.progressBar.clientWidth - progressBtnWidth
      let offsetWidth = newpercent * barWidth
      this.$refs.progress.style.width = `${offsetWidth}px`
      this.$refs.progressBtn.style[transform] = `translate3d(${offsetWidth}px, 0, 0)`
    }
  },
  methods: {
  }
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
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