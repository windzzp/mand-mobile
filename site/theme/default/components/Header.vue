<template>
  <div class="mfe-blog-theme-default-header" :class="{active: isActive}">
    <div class="default-header-container">
      <div class="default-header-aside">
        <a class="default-header-logo" href="/mand-mobile">
          <img :src="logo" alt="logo">
          <span v-html="title"></span>
        </a>
      </div>
      <div class="default-header-content">
        <div class="default-header-search">
          <div class="default-header-search-input">
            <i class="icon-magnifier"></i>
            <input
              type="text"
              placeholder="搜索"
              autocomplete="off"
              @click.stop
              @focus="onInputFocus($event)"
              @keyup="onInputKeyup($event)">
            <mfe-table v-model="searchTableShow" :data="searchData" style="top:45px;"></mfe-table>
          </div>
        </div>
        <div class="default-header-version">
          <div class="current-version" @click.stop="versionTableShow = true">
            <span>1.x</span>
          </div>
          <mfe-table v-model="versionTableShow" :data="versionData" style="width:96px;top:47px;left:auto !important;right:-8px;"></mfe-table>
        </div>
        <div class="default-header-nav">
          <ul>
            <li class="nav-item">
              <router-link :to="{path:'/home'}">首页</router-link>
            </li>
            <li class="nav-item" v-for="(item, index) in nav" :key="index">
              <template v-if="item.src && ~item.src.indexOf('//')">
                <a :href="item.src" v-html="item.text" target="_blank"></a>
              </template>
              <template v-else>
                <router-link
                  :to="`/${item.name}`"
                  v-html="item.text">
                </router-link>
              </template>
            </li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import MfeTable from './Table'
import algoliasearch from 'algoliasearch'

export default {
  name: 'mfe-blog-theme-default-header',

  components: {
    MfeTable
  },

  props: {
    isActive: {
      type: Boolean,
      default: false
    }
  },

  data() {
    return {
      nav: window.mbConfig.source,
      title: window.mbConfig.title,
      logo: window.mbConfig.logo,
      searchTableShow: false,
      searchData: [],
      versionTableShow: false,
      versionData: [{
        text: '1.x',
        path: '/home'
      }],
      searcher: algoliasearch('4GDUUWIAWB', 'd58846e82b7f4adfc81a0ada6346343f').initIndex('mand'),
      searchHandler: null
    }
  },

  methods: {
    onInputFocus(event) {
      const word = event.target.value.trim()
      if (word) {
        this.searchTableShow = true
      }
    },
    onInputKeyup(event) {
      const word = event.target.value.trim()

      if (this.searchHandler) {
        clearTimeout(this.searchHandler)
      }

      this.searchHandler = setTimeout(() => {
        this.preSearch(word)
        this.searchHandler = null
      }, 500)
    },
    preSearch(word) {
      if (word) {
        this.searchData = []
        this.createSearchData(word, this.nav)
        this.searchTableShow = true
      } else {
        this.searchTableShow = false
      }

    },
    matchIndex(index) {
      let matched
      if (index.component) {
        matched= this.traverseSource(index.component)
      } else {
        matched= this.traverseSource(index.name)
      }
      return {
        ...index,
        ...matched
      }
    },
    createSearchData(word) {
      if (word) {
        this.searcher.search({
          query: word,
          hitsPerPage: 10
        }, (err, res) => {
          if (err) {
            return
          }
          this.searchData = res.hits.map(this.matchIndex).map(this.handleSearchData)
        })
      }
    },
    handleSearchData(hit) {
      let content = hit._highlightResult.content.value.replace(/\s+/g, ' ')
      const highlightStart = content.indexOf('<em>')
      if (highlightStart > -1) {
        const startEllipsis = highlightStart - 25 > 0
        content = (startEllipsis ? '...' : '') +
          content.slice(Math.max(0, highlightStart - 15), content.length)
      } else if (content.indexOf('|') > -1) {
        content = ''
      }
      hit.content = content

      const matchedWord = hit._highlightResult.component.matchedWords[0]
      const componentNameMatchStart = (matchedWord && hit.text) ? hit.text.toLowerCase().indexOf(matchedWord.toLowerCase()) : -1

      if (componentNameMatchStart >= 0) {
        const myMatchedWord = hit.text.substr(componentNameMatchStart, matchedWord.length)
        hit.text = hit.text.replace(myMatchedWord, `<em>${myMatchedWord}</em>`)
      }

      return hit
    },

    traverseSource(word, source = this.nav, path = [], level = 0) {
      for (let i = 0, len = source.length; i < len; i++) {
        const item = source[i]
        const name = item.name

        path[level] = name
        if (item.menu && Array.isArray(item.menu)) {
          const tmpPath = [...path]
          level++
          const subItem = this.traverseSource(word, item.menu, path, level)
          if (subItem) {
            return subItem
          }
          path = [...tmpPath]
          level--
        }

        if (word === item.name) {
          item.path = `/${path.join('/')}`
          return item
        }
      }
    },

  },
}

</script>

<style lang="stylus">
.mfe-blog-theme-default-header
  position relative
  z-index 2
  display flex
  height 64px
  border-bottom solid 1px #f0f1f2
  box-shadow 0 2px 8px #f0f1f2
  clearfix()
  .default-header-container
    position relative
    display inline-block
    width 100%
    margin 0 auto
    transition width,height .5s,.5s
  .default-header-aside
    display inline-block
    width 16.666%
    height 100%
    padding-left 40px
    box-sizing border-box
    .default-header-logo
      float left
      height 100%
      text-decoration none
      overflow hidden
      line-height 64px
      img
        display inline-block
        margin-right 10px
        width auto
        height 28px
        vertical-align middle
        transition all .3s
      span
        height 100%
        color #333
        font-size 16px
        line-height 64px
        font-family AvenirNext-Medium,"Microsoft Yahei","Lato","Lucida Grande","Lucida Sans Unicode","Lucida Sans",Verdana,Tahoma,sans-serif  !important
        transition all .3s
        i
          color #048EFA
          font-size 13px
          font-style normal
          font-weight 500
          font-family "SFMono-Regular", Consolas, "Liberation Mono", Menlo, Courier, monospace  !important

  .default-header-content
    display inline-block
    width 83%
    height 100%
    padding-right 40px
    box-sizing border-box
    .default-header-search
      position relative
      float left
      height 22px
      margin-top 20px
      padding-left 16px
      line-height 22px
      // border-left 1px solid #ebedee
      i
        color #ccc
      input
        margin 0 10px
        width 200px
        outline none
        border none
        color #666
        font-size 14px

    .default-header-nav
      float right
      height 100%
      ul, li
        float left
        height 100%
      li.nav-item
        a
          position relative
          display inline-block
          height 100%
          padding 0 20px
          line-height 64px
          text-decoration none
          color #666
          font-weight 500
          transition color .3
          &:after
            display none
            content ""
            position absolute
            bottom 0
            left 30%
            width 40%
            height 4px
            background-color #048EFA
          &:hover
            color #333
          &.router-link-active
            color #048EFA
            &:after
              display block
    .default-header-version
      position relative
      float right
      width 80px
      height 30px
      margin-left 20px
      margin-top 17px
      line-height 30px
      text-align center
      border-radius 30px
      border solid 1px #ddd
      cursor pointer
      transition border .3s
      &:hover
        border-color #048EFA
      .current-version
        i
          font-size 12px
          color #048EFA
  &.active
    height 100px
    box-shadow none
    border none
    .default-header-container
      width 1280px
      .default-header-logo
        line-height 100px
        img
          height 48px
        span
          line-height 100px
          font-size 20px
      .default-header-nav
        li.nav-item a
          color #FFF !important
          line-height 100px
          &:after
            background-color #FFF
            bottom 20px
      .default-header-version
        margin-top 35px
        border-color #FFF
        color #FFF
      .default-header-aside
        padding 0
      .default-header-search
        display none
@media (max-width: 1400px)
  .default-header-container
    width 1080px !important
    .default-header-aside
      width 20% !important
    .default-header-content
      width 79% !important
      padding 0 !important
@media (max-width: 1200px)
  .default-header-container
    width 100% !important
  .default-header-logo span, .default-header-search
    display none
  .default-header-content
    width 80% !important
@media (max-width: 750px)
  .mfe-blog-theme-default-header
    display block
    height auto !important
    .default-header-aside, .default-header-content
      float left
      width 100% !important
      // height .64rem
      display flex
      justify-content center
      padding 0
    .default-header-nav li.nav-item a
      line-height .64rem !important
      &.router-link-active:after
        height .04rem
    .default-header-aside .default-header-logo
      height .64rem
      line-height .64rem !important
      img
        height .6rem !important
        margin 0
    .default-header-logo
      margin-top 10px
    .default-header-version
      display none
</style>
