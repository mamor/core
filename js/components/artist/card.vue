<template>
  <article class="item" v-if="showing" draggable="true" @dragstart="dragStart">
    <span class="cover" :style="{ backgroundImage: `url(${image})` }">
      <a class="control control-play" @click.prevent="playOrQueue">
        <i class="fa fa-play"></i>
      </a>
    </span>
    <footer>
      <div class="info">
        <a class="name" :href="`#!/artist/${artist.id}`">{{ artist.name }}</a>
      </div>
      <p class="meta">
        <span class="left">
          {{ artist.albums.length | pluralize('album') }}
          •
          {{ artist.songs.length | pluralize('song') }}
          •
          {{ artist.playCount | pluralize('play') }}
        </span>
        <span class="right">
          <a href @click.prevent="shuffle" title="Shuffle" class="shuffle-artist">
            <i class="fa fa-random"></i>
          </a>
          <a href @click.prevent="download" v-if="sharedState.allowDownload" title="Download all songs by artist" class="download-artist">
            <i class="fa fa-download"></i>
          </a>
        </span>
      </p>
    </footer>
  </article>
</template>

<script>
import { orderBy } from 'lodash'
import { pluralize } from '@/utils'
import { artistStore, queueStore, sharedStore } from '@/stores'
import { playback, download } from '@/services'
import artistAttributes from '@/mixins/artist-attributes'

export default {
  props: {
    artist: {
      type: Object,
      required: true
    }
  },
  filters: { pluralize },
  mixins: [artistAttributes],

  data: () => ({
    sharedState: sharedStore.state
  }),

  computed: {
    /**
     * Determine if the artist item should be shown.
     * We're not showing those without any songs, or the special "Various Artists".
     *
     * @return {Boolean}
     */
    showing () {
      return this.artist.songs.length && !artistStore.isVariousArtists(this.artist)
    }
  },

  methods: {
    playOrQueue (e) {
      if (e.metaKey || e.ctrlKey) {
        queueStore.queue(orderBy(this.artist.songs, ['album_id', 'disc', 'track']))
      } else {
        playback.playAllByArtist(this.artist, false)
      }
    },

    shuffle () {
      playback.playAllByArtist(this.artist, true /* shuffled */)
    },

    download () {
      download.fromArtist(this.artist)
    },

    dragStart (e) {
      const songIds = this.artist.songs.map(song => song.id)
      e.dataTransfer.setData('application/x-koel.text+plain', songIds)
      e.dataTransfer.effectAllowed = 'move'

      // Set a fancy drop image using our ghost element.
      const ghost = document.getElementById('dragGhost')
      ghost.innerText = `All ${pluralize(songIds.length, 'song')} by ${this.artist.name}`
      e.dataTransfer.setDragImage(ghost, 0, 0)
    }
  }
}
</script>

<style lang="scss">
@import "~#/partials/_vars.scss";
@import "~#/partials/_mixins.scss";

@include artist-album-card();
</style>
