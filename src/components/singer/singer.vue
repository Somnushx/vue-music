<template>
  <div class="singer" ref="singer">
      <list-view @select="selectSinger" :data="singerLists" ref="list"></list-view>
      <router-view></router-view>
  </div>
</template>

<script type="text/ecmascript-6">
import {getSingerList} from 'api/singer'
import {ERR_OK} from 'api/config'
import Singer from 'common/js/singer'
import ListView from 'base/listview/listview'
import {mapMutations} from 'vuex'
import {playlistMixin} from 'common/js/mixin'

const HOT_SINGER_LEN = 10
const HOT_NAME = '热门'
export default {
  mixins: [playlistMixin],
  data() {
    return {
      singerLists: []
    }
  },
  created() {
    this._getSingerList()
  },
  components: {
    ListView
  },
  methods: {
    handlePlaylist(playlist) {
      const bottom = playlist.length > 0 ? '60px' : ''
      this.$refs.singer.style.bottom = bottom
      this.$refs.list.refresh()
    },
    _getSingerList() {
      getSingerList().then(res => {
        if (res.code === ERR_OK) {
          this.singerLists = this._normalizeSingerList(res.data.list)
        }
      })
    },
    _normalizeSingerList(list) {
      let normalizeSingerList = {
        hot: {
          title: HOT_NAME,
          items: []
        }
      }
      list.forEach((item, index) => {
        if (index < HOT_SINGER_LEN) {
          normalizeSingerList.hot.items.push(new Singer({
            id: item.Fsinger_mid,
            name: item.Fsinger_name
          }))
        }
        let key = item.Findex
        if (!normalizeSingerList[key]) {
          normalizeSingerList[key] = {
            title: key,
            items: []
          }
        }
        normalizeSingerList[key].items.push(new Singer({
          id: item.Fsinger_mid,
          name: item.Fsinger_name
        }))
      })
      let hot = []
      let remain = []
      for (let key in normalizeSingerList) {
        let val = normalizeSingerList[key]
        if (val.title.match(/[a-zA-Z]/)) {
          remain.push(val)
        } else if (val.title === HOT_NAME) {
          hot.push(val)
        }
      }
      remain.sort((a, b) => {
        return a.title.charCodeAt(0) - b.title.charCodeAt(0)
      })
      return hot.concat(remain)
    },
    selectSinger(singer) {
      this.$router.push({
        path: `/singer/${singer.id}`
      })
      this.setSinger(singer)
    },
    ...mapMutations({
      setSinger: 'SET_SINGER' // 将this.setSinger()映射成为this.$store.commit('SET_SINGER')
    })
  }
}
</script>

<style scoped lang="stylus" rel="stylesheet/stylus">
.singer
  position: fixed
  top: 88px
  bottom: 0
  width: 100%
</style>