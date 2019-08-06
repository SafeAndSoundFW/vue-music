<template>
  <div class="singer">歌手页面</div>
</template>

<script type="text/ecmascript-6">
import { getSingerList } from 'api/singer'
import { ERR_OK } from 'api/config'
import Singer from 'common/js/singer'
const HOT_SINGER_LEN = 10
const HOT_NAME = "热门"

export default {
  data() {
    return {
      singers: []

    }
  },
  created() {
    this._getSingerList()
  },
  methods: {
    _getSingerList() {
      getSingerList().then(res => {
        if (res.code === ERR_OK) {
          console.log(res.data)
          this.singers = res.data.list
          this._normalizeSinger(this.singers)
        }
      })

    },
    _normalizeSinger(list) {
      let map = {
        hot: {
          title: HOT_NAME,
          items: []
        }
      }
      list.forEach((item, index) => {
        if (index < HOT_SINGER_LEN) {
          map.hot.items.push(new Singer({
            id: item.Fsinger_mid,
            name: item.Fsinger_name
          }))

          const key = item.Findex
          if (!map[key]) {
            map[key] = {
              title:key,
              items:[]
            }
          }
          map[key].items.push(new Singer({
            id: item.Fsinger_mid,
            name: item.Fsinger_name
          }))
        }
      })
     console.log(map)
    }

  }
}

</script>

<style scoped lang="stylus" rel="stylesheet/stylus"></style>