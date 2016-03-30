<style src="../assets/modal.sass" lang="sass?indentedSyntax"></style>
<style scoped lang="sass?indentedSyntax">
@import "../assets/variables.sass"
.new-download
  .content
    .label
      margin-bottom: 5px
      font-weight: 400
    .uris
      height: 100px
      white-space: nowrap
    .left-part
      float: left
      width: 68%
      padding-right: 12px
    .right-part
      float: right
      width: 32%
    .torrents-list
      overflow: auto
      white-space: nowrap
      min-height: 100px
      .torrent

</style>

<template lang="jade">
modal.new-download(:showing.sync="showing")
  .header
    .container NEW DOWNLOADS
    .close(@click="showing = false")
  .content
    .container
      .form-group(v-if="!torrents.length")
        .label LINK(S)
        textarea.uris(v-el:uri, placeholder="HTTP/FTP/SFTP/BitTorrent URIs", spellcheck="false", v-model="uris")
      .form-group(v-if="torrents.length")
        .label TORRENTS
        .torrents-list
          .torrent(v-for="torrent in torrents") {{torrent.name}}
      .group
        .left-part
          .label DESTINATION
          input(v-model="destination", spellcheck="false")
        .right-part
          .label CONNECTIONS
          input(v-model="connections", type="number")
  .bottom
    .container
      btn(:disabled="!allowDownload", @click="addDownloads") Download
</template>

<script>
import btn from './btn.vue'
import modal from './modal.vue'
import {trim} from 'lodash'

export default {
  data () {
    return {
      uris: '',
      connections: 5
    }
  },
  props: {
    showing: Boolean,
    destination: String,
    torrents: Array
  },
  watch: {
    'showing': function (newVal) {
      if (!newVal) return
      if (this.$els.uri) this.$els.uri.focus()
    }
  },
  computed: {
    uriArray: function () {
      return trim(this.uris, '\n').split('\n').map(trim).filter(uri => !!uri)
    },
    allowDownload: function () {
      return this.uriArray.length || this.torrents.length
    }
  },
  methods: {
    reset: function () {
      this.uris = ''
      this.torrents = []
    },
    addDownloads: function () {
      if (!this.allowDownload) return
      if (this.torrents.length) {
        this.addTorrentDownloads(this.torrents)
      } else {
        this.addUriDownloads(this.uriArray)
      }
      this.showing = false
      this.reset()
    },
    addUriDownloads: function (uris) {
      this.$dispatch('addUriDownloads', {
        uris: uris,
        options: {
          'max-connection-per-server': this.connections,
          'dir': this.destination
        }
      })
    },
    addTorrentDownloads: function (torrents) {
      this.$dispatch('addTorrentDownloads', {
        torrents: torrents,
        options: {
          'max-connection-per-server': this.connections,
          'dir': this.destination
        }
      })
    }
  },
  components: {
    btn,
    modal
  }
}
</script>