<template>
  <div class='search-list' v-show="searches.length">
    <transition-group name="list" tag="ul">
      <li class="search-item" @click="select(item)" v-for="item in searches" :key="item">
        <span class="title">{{item}}</span>
        <span class="icon" @click.stop="deleteOne(item)">
          <i class="icon-delete"></i>
        </span>
      </li>
    </transition-group>
  </div>
</template>

<script type='text/ecmascript-6'>
export default {
  data() {
    return {

    }
  },
  props: {
    searches: {
      type: Array,
      default: () => {}
    }
  },
  methods: {
    select(item) {
      this.$emit('select', item)
    },
    deleteOne(item) {
      this.$emit('delete', item)
    }
  }
}
</script>

<style scoped lang='stylus' rel='stylesheet/stylus'>
@import '~common/stylus/variable'

.search-list
  .search-item
    display: flex
    align-items: center
    height: 40px
    overflow: hidden
    &.list-enter-active, &.list-leave-active
      transition: all 0.1s
    &.list-enter, &.list-leave-to
      height: 0
    .title
      flex: 1
      color: $color-text-l
    .icon
      extend-click()
      .icon-delete
        font-size: $font-size-small
        color: $color-text-d
</style>