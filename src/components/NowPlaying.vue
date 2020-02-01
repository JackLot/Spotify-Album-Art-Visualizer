<template lang="pug">
.now-playing
  img(class="background" v-if="image.length > 0 && (albumArtVisible || alwaysShowAlbumArt) && !settingsVisible && !toast.visible && !hideAll" :src="image")
  .album-holder
    transition(name="fadeslide")
      img(class="cover" v-if="image.length > 0 && (albumArtVisible || alwaysShowAlbumArt) && !settingsVisible && !toast.visible && !hideAll" :src="image")
    .details
      transition(name="fade")
        div(:class="{ transitioning, initialized }" v-if="!settingsVisible && !toast.visible && !hideAll && (trackInfoVisible || alwaysShowTrackInfo)")
          span.album
            span(v-if="name" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") {{ album }}
          br
          span.artist
            span(v-if="artist" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") {{ artist }}
  span.name
    span(v-if="name" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") Album type: <strong>{{ album_type }}</strong><br />
    span(v-if="name" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") Album release: <strong>{{ album_release_date }}</strong><br />
    span(v-if="name" :style="{ transition: `opacity ${colorTransitionDuration}ms` }") Currently playing track {{ curr_track_num }} of {{ total_tracks }}: <strong>{{ name }}</strong>
</template>

<script>
import { mapState} from 'vuex'
import { SET_ALBUM_ART_VISIBLE, SET_TRACK_INFO_VISIBLE } from '@/vuex/mutation-types'

export default {
  data () {
    return {
      delay: 4000, // Visibility duration (ms)
      transitioning: false,
      initialized: false,
      transitionendIndex: 0,
      image: '',
      name: '',
      artist: '',
      album: '',
      curr_track_num: '',
      total_tracks: '',
      album_release_date: '',
      album_type: ''
    }
  },
  computed: {
    ...mapState([
      'color',
      'colorTransitionDuration',
      'currentlyPlaying',
      'albumArtVisible',
      'alwaysShowAlbumArt',
      'trackInfoVisible',
      'alwaysShowTrackInfo',
      'toast',
      'settingsVisible',
      'hover',
      'hideAll'
    ])
  },
  watch: {
    currentlyPlaying: {
      handler (val, old) {
        const values = this.getCurrentlyPlaying(val)
        this.image = values.image
        this.artist = values.artist
        this.name = values.name
        this.album = values.album
        this.curr_track_num = values.curr_track_num;
        this.total_tracks = values.total_tracks;
        this.album_release_date = values.album_release_date;
        this.album_type = values.album_type;
        this.show(val, old)
        // this.transition(values)
      },
      immediate: true
    }
  },
  methods: {
    show (val, old) {
      this.$store.commit(SET_ALBUM_ART_VISIBLE, true)
      this.$store.commit(SET_TRACK_INFO_VISIBLE, true)

      if (!old) return

      this.timeout = setTimeout(() => {
        if (this.alwaysShowAlbumArt === false) {
          this.$store.commit(SET_ALBUM_ART_VISIBLE, false)
        }

        if (this.alwaysShowTrackInfo === false) {
          this.$store.commit(SET_TRACK_INFO_VISIBLE, false)
        }
      }, this.delay)
    },
    getCurrentlyPlaying (val = this.currentlyPlaying) {
      const name = val.name || false
      const album_deets = val.album
      const album = album_deets.name
      const image = album_deets ? album_deets.images[1].url : false
      const artist = album_deets ? album_deets.artists[0].name : ''
      const curr_track_num = val.track_number;
      const total_tracks = album_deets ? album_deets.total_tracks : 'NA'
      const album_release_date = album_deets ? album_deets.release_date : 'NA'
      const album_type = album_deets ? album_deets.album_type : 'NA'
      return { name, image, artist, album, curr_track_num, total_tracks, album_release_date, album_type }
    }
  }
}
</script>

<style lang="scss" scoped>
body {
  background-color: black;
}
.now-playing {
  @include position(absolute, 0 0 0 0);
  @include flex(center, center);
  z-index: 200;
  text-align: left;
  width: 100%;
  height: 100%;
}

.album-holder {
  // width: 60%;
  margin-left: 350px;
  display: flex;
  align-items: flex-end;
}

.details {
  width: 400px;
  max-width: 400px;
  margin-bottom: 2px;
}

.background {
  position: absolute;
  z-index: -1;
  object-fit: cover;
  left: -25px;
  top: -25px;
  width: calc(100% + 50px);
  height: calc(100% + 50px);
  -webkit-filter: blur(30px) brightness(60%);
  overflow: hidden;
}

img.cover {
  height: 380px;
  width: 380px;
  margin-right: 15px;
  box-shadow: 0 4px 33px 5px rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

.album {
  @include share;
  @include scale(font-size 32px 18px);
}

.artist {
  @include scale(font-size 20px 16px);
}

.name, .artist, .album {
  position: relative;

  span {
    background: black;
    color: white;
    padding: 3px 6px;
    position: relative;
    z-index: 10;

    // opacity: 0;
  }
}

.name {
  @include scale(font-size 17px 13px);
  position: absolute;
  left: 10px;
  bottom: 10px;
  span {
    background: none;
    opacity: 0.7;
  }
}

i {
  @include position(absolute, 0 0 0 0);
  transform: scaleX(0);
  z-index: 20;
  display: block;
  background: #ffffff;
  transform-origin: left;
}

.initialized {
  .name, .artist, .album {
    span { opacity: 1; }
  }
}

.transitioning i { transform: scaleX(1); }
</style>
