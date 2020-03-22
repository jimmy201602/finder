<template>
  <q-layout view="hHh lpR fFf">

    <q-header elevated class="bg-primary text-white">
      <q-toolbar>
        <q-btn dense flat round icon="menu" @click="left = !left"/>
<!--        <q-toolbar-title>-->
<!--          <q-avatar>-->
<!--            <img src="https://cdn.quasar.dev/logo/svg/quasar-logo.svg">-->
<!--          </q-avatar>-->
<!--          Webterminal-->
<!--        </q-toolbar-title>-->
        <breadcrumbs
          :absolutePath="selectedFolder"
          @selected="onSelectedFolder"
        />
        <div class="float-right" style="margin-left: 10px;">
          <q-btn
            flat
            dense
            round
            aria-label="toggle between grid and list modes"
            @click="toggleListType"
          >
            <q-icon :name="listType === 'grid' ? 'format_list_bulleted' : 'border_all'" />
          </q-btn>
        </div>
        <div class="YL__toolbar-input-container row no-wrap">
          <q-input dense outlined square v-model="search" :placeholder="searchPlaceholder" @focus="onSearchFocus"  @blur="onSearchBlur" class="bg-white col" />
          <q-btn class="YL__toolbar-input-btn" color="grey-3" text-color="grey-8" icon="search" unelevated />
        </div>
      </q-toolbar>
    </q-header>

    <q-drawer v-model="left" side="left" behavior="desktop" bordered>
      <q-scroll-area class="fit">
        <shortcuts @selected="onShortcutSelected"/>
        <folder-tree
          :rootDir="rootDir"
          :folder.sync="selectedFolder"
          :lazyLoad="onLazyLoad"
          @selected="onSelectedFolder"
        />
      </q-scroll-area>
    </q-drawer>

    <q-page-container>
      <contents
        :contents="contents"
        :listType="listType"
        :viewType="viewType"
        @click="onClicked"
        @dblClick="onDblClicked"
      />
    </q-page-container>

  </q-layout>
</template>

<script>
// identify mime types
const mime = require('mime-types')
const path = require('path')
import walkFolders from '../util/walkFolders'
const fs = require('fs')
// file watcher
// const chokidar = require('chokidar')
export default {
  created () {
    this.setSelectedFolder(this.drive + path.sep)
    this.rootDir.push(...this.getFolders(this.selectedFolder))
    // this.$root.$on('rescan-current-folder', this.rescanCurrentFolder)
    this.rootDir = require('../mock/mockdata').default
  },
  data () {
    return {
      contents: [], // children of a node
      viewType: 'nodes',
      listType: 'grid', // default ['grid', 'list']
      left: true,
      drive: '',
      search: '',
      selectedFolder: null, // the selected node (label)
      rootDir: [], // tree view
      toolbarLinks: [], // toolbar pathway (links to each folder in path)
      drives: [],
      watcher: null
    }
  },
  beforeDestroy: function () {
    this.$root.$off('rescan-current-folder', this.rescanCurrentFolder)
  },
  watch: {
    selectedFolder: function (newFolder, oldFolder) {
      // The User can de-select a folder, in which case
      // value will be null, so use root folder
      if (!newFolder) {
        newFolder = this.drive + path.sep
      }

      // folder watcher handler
      // this.folderWatcherHandler(newFolder, oldFolder)

      this.clearAllContentItems()
      // this.contents.push(...this.getFolderContents(newFolder))
      this.contents = require('../mock/contents').default
    }
  },
  methods: {
    // folderWatcherHandler: function (newFolder, oldFolder) {
    //   if (oldFolder && this.watcher) {
    //     this.watcher.close()
    //   }
    //   if (newFolder) {
    //     // let backend know to statically serve files from this folder
    //     this.watcher = chokidar.watch(newFolder, {
    //       depth: 0,
    //       ignorePermissionErrors: true
    //     })
    //     if (this.watcher) {
    //       this.watcher.on('ready', () => { // initial scan done
    //         // watch for additions
    //         this.watcher.on('raw', (event, path, details) => {
    //           this.$root.$emit('rescan-current-folder')
    //         })
    //       })
    //       this.watcher.on('error', (error) => { // initial scan done
    //         console.error(error)
    //       })
    //     }
    //   }
    // },
    checkForDrive: async function (drives, index) {
      return new Promise(function (resolve, reject) {
        try {
          const stat = fs.statSync(drives[index] + ':' + path.sep)
          drives[index] += ':\\'
          resolve(stat)
        } catch (error) {
          // remove from drives list
          drives.splice(index, 1)
          reject(error)
        }
      })
    },
    rescanCurrentFolder: function () {
      this.clearAllContentItems()
      this.contents.push(...this.getFolderContents(this.selectedFolder))
    },
    setFolder: function (folder) {
      this.selectedFolder = folder
    },
    getFolders: function (absolutePath) {
      const folders = []

      // check incoming arg
      if (!absolutePath || typeof absolutePath !== 'string') {
        return folders
      }

      for (const fileInfo of walkFolders(absolutePath, 0)) {
        // all files and folders
        if ('error' in fileInfo) {
          console.error(`Error: ${fileInfo.rootDir} - ${fileInfo.error}`)
          continue
        }
        // we only want folders
        if (!fileInfo.isDir) {
          continue
        }
        const node = this.createNode(fileInfo)
        folders.push(node)
      }
      return folders
    },
    getFolderContents: function (folder) {
      const contents = []

      // check incoming arg
      if (!folder || typeof folder !== 'string') {
        return contents
      }

      for (const fileInfo of walkFolders(folder, 0)) {
        // all files and folders
        if ('error' in fileInfo) {
          console.error(`Error: ${fileInfo.rootDir} - ${fileInfo.error}`)
          continue
        }
        const node = this.createNode(fileInfo)
        contents.push(node)
      }

      return contents
    },
    clearAllContentItems: function () {
      this.contents.splice(0, this.contents.length)
    },
    addContentItem: function (item) {
      this.contents.push(item)
    },
    onClicked: function (node) {
      // on single-clicks we don't do anything here
      // if we wanted to drill-down into folders, we
      // can call this.onDblClicked function.
    },

    onDblClicked: function (node) {
      // This causes a drill-down if it's a folder
      if (node.data.isDir) {
        this.setSelectedFolder(node.nodeKey)
      } else {
        this.onFileSelected(node)
      }
    },
    onFileSelected: function (node) {
      console.log(node.label)
    },
    toggleListType: function () {
      if (this.listType === 'grid') {
        this.listType = 'list'
      } else {
        this.listType = 'grid'
      }
    },
    onSelectedFolder: function (absolutePath) {
      this.setSelectedFolder(absolutePath)
      const data = require('../mock/mockdata').default
      var min = Math.ceil(1)
      var max = Math.floor(data.length)
      var random = Math.floor(Math.random() * (max - min + 1)) + min
      // this.rootDir = data.slice(0, random)
      this.contents = require('../mock/contents').default.slice(0, random)
    },
    // called by folderTree component
    onLazyLoad: function ({ node, key, done, fail }) {
      if (this.loadChildren(node, key)) {
        done()
      } else {
        // if we don't call done, then the tree will
        // allow user to try and expand node again
        done()
      }
    },
    loadChildren: function (node, key) {
      try {
        if (node.children.length) {
          node.children.splice(0, node.children.length)
        }
        for (const fileInfo of walkFolders(key, 0)) {
          // we only want folders
          if (!fileInfo.isDir) {
            continue
          }
          // create a new node
          const n = this.createNode(fileInfo)
          // add child to parent
          node.children.push(n)
        }
        return true
      } catch (err) {
        // usually access error
        console.error('Error: ', err)
      }
      return false
    },
    setSelectedFolder: function (folder) {
      this.selectedFolder = folder
    },
    onShortcutSelected: function (type) {
      console.log('onShortcutSelected type:', type)
      // let absolutePath = app.getPath(type)
      const absolutePath = '/'
      this.setSelectedFolder(absolutePath)
    },
    focusOnSearch (evt) {
      if (
        evt.target.tagName !== 'INPUT' &&
        String.fromCharCode(evt.keyCode) === '/'
      ) {
        evt.preventDefault()
        this.search = ''
        if (!this.leftDrawerState) {
          this.leftDrawerState = true
        }
        setTimeout(() => {
          this.$refs.docAlgolia.focus()
        })
      }
    },
    onSearchFocus () {
      this.searchFocused = true
    },
    onSearchBlur () {
      this.searchFocused = false
    },
    createNode: function (fileInfo) {
      let nodeKey = fileInfo.rootDir
      if (nodeKey.charAt(nodeKey.length - 1) !== path.sep) {
        nodeKey += path.sep
      }
      if (fileInfo.fileName === path.sep) {
        fileInfo.fileName = nodeKey
      } else {
        nodeKey += fileInfo.fileName
      }
      // get file mime type
      const mimeType = mime.lookup(nodeKey)
      // create object
      return {
        label: fileInfo.fileName,
        nodeKey: nodeKey,
        expandable: fileInfo.isDir,
        tickable: true,
        lazy: true,
        children: [],
        data: {
          rootDir: fileInfo.rootDir,
          isDir: fileInfo.isDir,
          mimeType: mimeType,
          stat: fileInfo.stat
        }
      }
    }
  },
  computed: {
    searchPlaceholder () {
      return this.searchFocused === true
        ? 'Type to start searching...'
        : (this.$q.platform.is.desktop === true ? 'Type \' / \' to focus here...' : 'Search...')
    }
  },
  components: {
    shortcuts: require('../components/shortcuts').default,
    // eslint-disable-next-line vue/no-unused-components
    'folder-tree': require('../components/folderTree').default,
    breadcrumbs: require('../components/breadcrumbs').default,
    contents: require('../pages/contents').default
  }
}
</script>
<style>
  .breadcrumbs {
    width: calc(100% - 150px);
    height: 20px;
    margin-left: 5px;
    background-color: rgba(0, 0, 0, .3);
    border-radius: 4px;
  }

  .breadcrumb {
    cursor: pointer;
  }

  .breadcrumb:hover {
    text-decoration: underline;
  }

  .border {
    border-style: solid;
    border-radius: 2px;
    border-width: 1px;
    border-color: rgb(158, 158, 158);
    background-color: rgba(158, 158, 158, 0.1);
  }

  .q-layout-drawer {
    height: calc(100vh - 50px) !important;
  }

  .drawer-wrapper {
    position: relative;
    width: 100%;
    height: calc(100vh - 50px) !important;
  }

  .drawer-container {
    position: relative;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    overflow: auto;
  }

</style>
<style lang="sass">
  .YL
    &__toolbar-input-container
      min-width: 20px
      width: 15%
    &__toolbar-input-btn
      border-radius: 0
      border-style: solid
      border-width: 1px 1px 1px 0
      border-color: rgba(0,0,0,.24)
      max-width: 60px
      width: 100%
    &__drawer-footer-link
      color: inherit
      text-decoration: none
      font-weight: 500
      font-size: .75rem
      &:hover
        color: #000
</style>
