<template>

		<transition name="slide">
				<!-- <div class="singer-detail">djkdsjfkdjfdksfkdsfdksj</div> -->
		  <music-list :songs="songs" :title="title" :bg-image="bgImage"></music-list>
		</transition>
	  </template>
	  
	  <script>
	//   singer-detail -> music-list ->scroll -> song-list 
		import {ERR_OK} from 'api/config'
		import {mapGetters} from 'vuex'
		import {getSingerDetail} from 'api/singer'
		import {createSong,processSongsUrl,isValidMusic} from 'common/js/song'
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
					getSingerDetail(this.singer.id).then((res) => {
							if (res.code === ERR_OK) {
								processSongsUrl(this._normalizeSongs(res.data.list)).then((songs) => {
								this.songs = songs
								})
							}
							})
						},
				   _normalizeSongs (list) {
						let ret = []
						list.forEach((item) => {
						let { musicData } = item
						if (isValidMusic(musicData)) {
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