<template>
  <q-list padding>
    <q-item>
      <q-item-section>
        <q-item-label>Shortcuts</q-item-label>
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onHome" v-ripple>
      <q-item-section avatar>
        <q-icon name="home"/>
      </q-item-section>
      <q-item-section>
        Home
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onDesktop" v-ripple>
      <q-item-section avatar>
        <q-icon name="desktop_windows"/>
      </q-item-section>
      <q-item-section>
        Desktop
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onDocuments" v-ripple>
      <q-item-section avatar>
        <q-icon name="folder"/>
      </q-item-section>
      <q-item-section>
        Documents
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onDownloads" v-ripple>
      <q-item-section avatar>
        <q-icon name="vertical_align_bottom"/>
      </q-item-section>
      <q-item-section>
        Downloads
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onPictures" v-ripple>
      <q-item-section avatar>
        <q-icon name="image"/>
      </q-item-section>
      <q-item-section>
        Pictures
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onMusic" v-ripple>
      <q-item-section avatar>
        <q-icon name="music_note"/>
      </q-item-section>
      <q-item-section>
        Music
      </q-item-section>
    </q-item>
    <q-item clickable @click.native="onMovies" v-ripple>
      <q-item-section avatar>
        <q-icon name="local_movies"/>
      </q-item-section>
      <q-item-section>
        Movies
      </q-item-section>
    </q-item>
  </q-list>
</template>

<script>
export default {
  name: 'Shortcuts',
  data () {
    return {}
  },

  methods: {
    onHome: function () {
      this.$emit('selected', 'home')
    },
    onDesktop: function () {
      this.$emit('selected', 'desktop')
    },
    onDocuments: function () {
      this.$emit('selected', 'documents')
    },
    onDownloads: function () {
      this.$emit('selected', 'downloads')
    },
    onPictures: function () {
      this.$emit('selected', 'pictures')
    },
    onMusic: function () {
      this.$emit('selected', 'music')
    },
    onMovies: function () {
      this.$emit('selected', 'videos')
    }
  }
}
</script>

<style>
</style>
