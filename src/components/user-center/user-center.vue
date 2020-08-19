<template>
  <transition name="slide">
    <div class="user-center">
      <div class="back" @click="back">
        <i class="icon-back"></i>
      </div>
      <div class="switch-wrapper">
        <switches
          @switch="switchItem"
          :switches="switches"
          :currentIndex="currentIndex"
        ></switches>
      </div>
      <div class="play-btn" ref="playBtn" @click="random">
        <i class="icon-play"></i>
        <span class="text">随机播放全部</span>
      </div>
      <div class="list-wrapper" ref="listWrapper">
        <scroll
          v-if="currentIndex === 0"
          :data="favoriteList"
          class="list-scroll"
          ref="favoriteList"
        >
          <div class="list-inner">
            <song-list :songs="favoriteList" @select="selectSong"></song-list>
          </div>
        </scroll>
        <scroll
          v-if="currentIndex === 1"
          :data="playHistory"
          class="list-scroll"
          ref="playList"
        >
          <div class="list-inner">
            <song-list :songs="playHistory" @select="selectSong"></song-list>
          </div>
        </scroll>
      </div>
      <div class="no-result-wrapper" v-show="noResult">
        <no-result :title="noResultDesc"></no-result>
      </div>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
import Switches from 'base/switches/switches'
import {mapGetters, mapActions} from 'vuex'
import Scroll from 'base/scroll/scroll'
import SongList from 'base/song-list/song-list'
import Song from 'common/js/song'
import {playlistMixin} from 'common/js/mixin'
import NoResult from 'base/no-result/no-result'

export default {
  mixins: [playlistMixin],
  data() {
    return {
      switches: [
        {
          name: '我喜欢的'
        },
        {
          name: '最近听的'
        }
      ],
      currentIndex: 0
    }
  },
  computed: {
    ...mapGetters([
      'favoriteList',
      'playHistory'
    ]),
    noResult() {
      if (this.currentIndex === 0) {
        return !this.favoriteList.length
      } else {
        return !this.playHistory.length
      }
    },
    noResultDesc() {
      if (this.currentIndex === 0) {
        return '暂无收藏歌曲'
      } else {
        return '你还没有听过歌曲'
      }
    }
  },
  methods: {
    handlePlaylist(playlist) {
      const bottom = playlist.length > 0 ? '60px' : ''
      this.$refs.listWrapper.style.bottom = bottom
      this.$refs.favoriteList && this.$refs.favoriteList.refresh()
      this.$refs.playList && this.$refs.playList.refresh()
    },
    back() {
      this.$router.back()
    },
    switchItem(index) {
      this.currentIndex = index
    },
    selectSong(song) {
      this.insertSong(new Song(song))
    },
    random() {
      let list = this.currentIndex === 0 ? this.favoriteList : this.playHistory
      if (list.length === 0) {
        return
      }
      list = list.map((song) => {
        return new Song(song)
      })
      this.randomPlay({list})
    },
    ...mapActions([
      'insertSong',
      'randomPlay'
    ])
  },
  components: {
    Switches,
    Scroll,
    SongList,
    NoResult
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
@import '~common/stylus/variable'

.user-center
  position: fixed
  bottom: 0
  top: 0
  width: 100%
  z-index: 100
  background: $color-background
  &.slide-enter-active, &.slide-leave-active
    transition: all 0.1s
  &.slide-enter, &.slide-leave-to
    transform: translate3d(100%, 0, 0)
  .back
    position: absolute
    left: 6px
    top: 0
    z-index: 50
    .icon-back
      display: block
      padding: 10px
      font-size: $font-size-large-x
      color: $color-theme
  .switch-wrapper
    margin: 10px 0 30px 0
  .play-btn
    width: 135px
    padding: 7px 0
    margin: 0 auto
    text-align: center
    color: $color-text-l
    border: 1px solid $color-text-l
    border-radius: 100px
    font-size: 0
    .icon-play
      vertical-align: middle
      margin-right: 6px
      font-size: 16px
    .text
      vertical-align: middle
      font-size: 12px
  .list-wrapper
    position: absolute
    top: 110px
    bottom: 0
    width: 100%
    .list-scroll
      height: 100%
      overflow: hidden
      .list-inner
        padding: 20px 30px
  .no-result-wrapper
    position: absolute
    width: 100%
    top: 50%
    transform: translateY(-50%)
</style>
