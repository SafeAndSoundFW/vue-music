<template>

		<transition name="slide">
				<div class="singer-detail">djkdsjfkdjfdksfkdsfdksj</div>
		  <!-- <music-list :title="title" :bg-image="bgImage" :songs="songs"></music-list> -->
		</transition>
	  </template>
	  
	  <script>
		import {ERR_OK} from 'api/config'
		import {mapGetters} from 'vuex'

		export default {
			computed:{
				...mapGetters([
					'singer'
				])
			},
			created() {
				console.log(this.singer)
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