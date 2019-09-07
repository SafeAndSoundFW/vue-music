<template>

		<transition name="slide">
				<!-- <div class="singer-detail">djkdsjfkdjfdksfkdsfdksj</div> -->
		  <music-list :songs="songs" :title="title" :bg-image="bgImage"></music-list>
		</transition>
	  </template>
	  
	  <script>
		import {ERR_OK} from 'api/config'
		import {mapGetters} from 'vuex'
		import {getSingerDetail} from 'api/singer'
		  import {createSong} from 'common/js/song'
		import MusicList from 'components/music-list/music-list'
		export default {
			computed:{
				title() {
					return this.singer.name
				},
				bgImage() {
					return this.singer.avatar
				},
				...mapGetters([
					'singer'
				])
			},
			created() {
				this._getDetail()
				console.log(this.singer)
			},
			   data() {
				return {
					songs: []
				}
			   },
			methods:{
			
				_getDetail() {
				   if (!this.singer.id) {
					    this.$router.back()
					   return
				    }
					getSingerDetail(this.singer.id).then(res => {
						if (res.code === ERR_OK) {
							
							 this.songs = this._normalizeSongs(res.data.list)
							 console.log( this.songs)
						}
					})

				},
				      _normalizeSongs(list) {
						let ret = []
						list.forEach((item) => {
						let {musicData} = item
						if (musicData.songid && musicData.albummid) {
							ret.push(createSong(musicData))
						}
						})
						return ret
					}
				
			},
			components:{
				 MusicList
			}
		}
	  </script>
	  
	  <style scoped lang="stylus" rel="stylesheet/stylus">
	  @import "~common/stylus/variable"
	
		.slide-enter-active, .slide-leave-active
		  transition: all 0.3s
	  
		.slide-enter, .slide-leave-to
		  transform: translate3d(100%, 0, 0)

		.singer-detail
		  position:fixed
		  top:0
		  left:0
		  right:0
		  bottom:0
		  z-index:100
		  background:$color-background
		  
	  </style>