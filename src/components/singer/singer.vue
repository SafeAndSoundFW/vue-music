<template>
  <div class="singer">歌手页面</div>
</template>

<script type="text/ecmascript-6">
import { getSingerList } from 'api/singer'
import { ERR_OK } from 'api/config'

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
          map.hot.items.push({
            id: item.Fsinger.mid,
            name: item.Fsinger.name,
            avatar: ''
          })
        }
      })

    }

  }
}

</script>

<style scoped lang="stylus" rel="stylesheet/stylus"></style>

