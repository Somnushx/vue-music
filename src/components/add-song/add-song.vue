<template>
  <transition name="slide">
    <div class="add-song" v-show="showFlag" @click.stop>
      <div class="header">
        <h1 class="text">添加歌曲到列表</h1>
        <span class="close" @click="hide">
          <i class="icon-close"></i>
        </span>
      </div>
      <div class="search-box-wrapper">
        <search-box
          ref="searchbox"
          placeholder="搜索歌曲"
          @query="onQueryChange"
        ></search-box>
      </div>
      <div class="shortcut" v-show="!query">
        <switches
          :switches="switches"
          :currentIndex="currentIndex"
          @switch="switchItem"
        ></switches>
        <div class="list-wrapper">
          <scroll
            v-if="currentIndex === 0"
            :data="playHistory"
            class="list-scroll"
            ref="songList"
          >
            <div class="list-inner">
              <song-list :songs="playHistory" @select="selectSong"></song-list>
            </div>
          </scroll>
          <scroll
            v-if="currentIndex === 1"
            :data="searchHistory"
            class="list-scroll"
            ref="searchList"
            :refreshDelay="refreshDelay"
          >
            <div class="list-inner">
              <search-list
                @delete="deleteSearchHistory"
                @select="addQuery"
                :searches="searchHistory"
              ></search-list>
            </div>
          </scroll>
        </div>
      </div>
      <div class="search-result" v-show="query">
        <suggest
          :query="query"
          :showSinger="showSinger"
          @select="selectSuggest"
          @listScroll="blurInput"
        ></suggest>
      </div>
      <top-tip ref="topTip">
        <div class="tip-title">
          <i class="icon-ok"></i>
          <span class="text">1首歌曲已经添加到播放队列</span>
        </div>
      </top-tip>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
import SearchBox from 'base/search-box/search-box'
import Suggest from 'components/suggest/suggest'
import {mapActions, mapGetters} from 'vuex'
import Switches from 'base/switches/switches'
import Scroll from 'base/scroll/scroll'
import SongList from 'base/song-list/song-list'
import Song from 'common/js/song'
import SearchList from 'base/search-list/search-list'
import TopTip from 'base/top-tip/top-tip'

export default {
  data() {
    return {
      showFlag: false,
      showSinger: false,
      query: '',
      switches: [
        {
          name: '最近播放'
        },
        {
          name: '搜索历史'
        }
      ],
      currentIndex: 0,
      refreshDelay: 100
    }
  },
  computed: {
    ...mapGetters([
      'playHistory',
      'searchHistory'
    ])
  },
  methods: {
    show() {
      this.showFlag = true
      setTimeout(() => {
        if (this.currentIndex === 0) {
          this.$refs.songList.refresh()
        } else {
          this.$refs.searchList.refresh()
        }
      }, 20)
    },
    hide() {
      this.showFlag = false
    },
    onQueryChange(query) {
      this.query = query
    },
    selectSuggest() {
      this.saveSearchHistory(this.query)
      this.showTip()
    },
    addQuery(query) {
      this.$refs.searchbox.setQuery(query)
    },
    blurInput() {
      this.$refs.searchbox.blur()
    },
    switchItem(index) {
      this.currentIndex = index
    },
    selectSong(song, index) {
      if (index !== 0) {
        this.insertSong(new Song(song))
      }
      this.showTip()
    },
    showTip() {
      this.$refs.topTip.show()
    },
    ...mapActions([
      'saveSearchHistory',
      'deleteSearchHistory',
      'insertSong'
    ])
  },
  components: {
    SearchBox,
    Suggest,
    Switches,
    Scroll,
    SongList,
    SearchList,
    TopTip
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable'

.add-song
  position: fixed
  width: 100%
  height: 100%
  background: $color-background
  z-index: 200
  &.slide-enter-active, &.slide-leave-active
    transition: all 0.3s
  &.slide-enter, &.slide-leave-to
    transform: translate3d(100%, 0, 0)
  .header
    position: relative
    height: 44px
    text-align: center
    .text
      line-height: 44px
      color: $color-text
      font-size: $font-size-large
    .close
      position: absolute
      right: 8px
      top: 0
      .icon-close
        display: block
        padding: 12px
        font-size: 20px
        color: $color-theme
  .search-box-wrapper
    margin: 20px
  .shortcut
    .list-wrapper
      position: absolute
      top: 165px
      bottom: 0
      width: 100%
      .list-scroll
        height: 100%
        overflow: hidden
        .list-inner
          padding: 20px 30px
  .search-result
    position: fixed
    top: 124px
    bottom: 0
    width: 100%
  .tip-title
    text-align: center
    padding: 18px 0
    font-size: 0
    .icon-ok
      font-size: $font-size-medium
      color: $color-theme
      margin-right: 4px
    .text
      font-size: $font-size-medium
      color: $color-text
</style>
