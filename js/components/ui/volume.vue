<template>
  <span class="volume control" id="volume">
    <i class="fa fa-volume-off unmute" @click.prevent="unmute" v-if="muted"></i>
    <i class="fa fa-volume-up mute" @click.prevent="mute" v-else></i>
    <input title="Volume" type="range" id="volumeRange" max="10" step="0.1"
      @change="broadcastVolume" class="plyr__volume"
      @input="setVolume"
    >
  </span>
</template>

<script>
import { playback, socket } from '@/services'
import { event } from '@/utils'

export default {
  data: () => ({
    muted: false
  }),

  methods: {
    mute () {
      this.muted = true
      playback.mute()
    },

    unmute () {
      this.muted = false
      playback.unmute()
    },

    setVolume (e) {
      const volume = parseFloat(e.target.value)
      playback.setVolume(volume)
      this.muted = volume === 0
    },

    /**
     * Broadcast the volume changed event to remote controller.
     *
     * @param  {Event} e
     */
    broadcastVolume: e => socket.broadcast(event.$names.SOCKET_VOLUME_CHANGED, parseFloat(e.target.value))
  }
}
</script>

<style lang="scss">
@import "~#/partials/_vars.scss";
@import "~#/partials/_mixins.scss";

#volume {
  @include vertical-center();

  // More tweaks
  input[type=range] {
    margin-top: -3px;
  }

  i {
    width: 16px;
  }

  @media only screen and (max-width: 768px) {
    display: none !important;
  }
}
</style>
