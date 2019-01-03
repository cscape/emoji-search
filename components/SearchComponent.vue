<template>
  <section class="search-app">
    <form v-on:submit="SubmitHost($event)">
      <div class="d-flex">
        <span class="h2 at">@</span>
        <input class="searchbox h2 d-block" ref="Searchbox" value="loading..."
          type="search"
          :placeholder="defaultUserSearchInput"
          v-model='UserSearchInput' @mousedown="ClickSearchBox"
          @blur="BlurSearchBox" @keypress="KeypressSearchBox($event)"/>
      </div>
      <input type="submit" hidden>
      <p class="error-box h4" v-show="UserSearchInputError">
        ⚠ <span class="pl-2" v-html="UserSearchInputError"></span>
      </p>
    </form>
    <div class="emoji-panel" v-if="loadedEmojis">
      <div class="emoji-search">
        <input class="searchbox h3" ref="Emojisearch"
          v-model="EmojiSearchInput"
          :placeholder="`Search ${loadedEmojis.length} emojis...`"
          @keyup="EmojiSearchInputChange"/>
      </div>
      <div class="emojis-box">
        <div class="emoji-search-results" v-if="EmojiSearchResults != null && IsEmojiSearchFilled">
          <recycle-list key-field="shortcode" :item-height="78"
            class="emojis-scroll container-fluid" :items="EmojiSearchResults">
            <template slot-scope="{ item, index, active }">
              <div class="emoji-row row">
                <div class="emoji-row-image-container col-auto">
                  <img :src="item.url" :style="{ display: active ? 'block' : 'none' }"
                    :alt="`:${item.shortcode}:`" height="auto"
                    width="auto" class="emoji-row-image"/>
                </div>
                <div class="emoji-row-header-container col">
                  <span class="h3 emoji-row-name">{{ item.shortcode }}</span>
                </div>
              </div>
            </template>
          </recycle-list>
        </div>
        <div class="emoji-search-error"
          v-if="EmojiSearchResults && EmojiSearchResults.length === 0">
          No emojis here.
        </div>
        <div :style="{ display: IsEmojiSearchFilled ? 'none': null }">
          <recycle-list ref="MainEmojis" key-field="shortcode" :item-height="78"
            class="emojis-scroll container-fluid" :items="loadedEmojis">
            <template slot-scope="{ item, index, active }">
              <div class="emoji-row row">
                <div class="emoji-row-image-container col-auto">
                  <img :src="item.url" :style="{ display: active ? 'block' : 'none' }"
                    :alt="`:${item.shortcode}:`" height="auto"
                    width="auto" class="emoji-row-image"/>
                </div>
                <div class="emoji-row-header-container col">
                  <span class="h3 emoji-row-name">{{ item.shortcode }}</span>
                </div>
              </div>
            </template>
          </recycle-list>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios'
import { RecycleList } from 'vue-virtual-scroller'
import Fuse from 'fuse.js'
const TimSort = require('timsort')

const defaultUserSearchInput = 'type in an instance...'

export default {
  components: { RecycleList },
  data () {
    return {
      defaultUserSearchInput,
      UserSearchInput: defaultUserSearchInput,
      UserSearchInputError: null,
      loadedEmojis: null,
      EmojiSearchInput: null,
      EmojiSearchResults: null
    }
  },
  computed: {
    IsUserSearchInputValid () {
      return this.UserSearchInput.match(/^([^/]){1,}\.([^/]){2,}$/g) !== null
    },
    IsEmojiSearchFilled () {
      return this.EmojiSearchInput != null ? this.EmojiSearchInput.trim().length > 1 : false
    }
  },
  mounted () {
    if (window.location.hash.indexOf('#') === 0) {
      const hash = window.location.hash.split('#')[1].toLowerCase()
      if (this.IsHostValid(hash)) {
        this.UserSearchInput = hash
        return this.SubmitHost()
      }
    }
    this.$refs.Searchbox.focus()
  },
  methods: {
    ClickSearchBox () {
      if (this.UserSearchInput === defaultUserSearchInput) 
        this.UserSearchInput = ''
    },
    BlurSearchBox () {
      if (this.UserSearchInput === '') this.UserSearchInput = defaultUserSearchInput
    },
    KeypressSearchBox ($event) {
      const _default = defaultUserSearchInput
      const inputOrig = this.UserSearchInput
      const input = this.UserSearchInput + $event.key
      if (-1 !== input.indexOf(_default) && input.length > _default.length) {
        $event.target.value = ''
        this.UserSearchInput = $event.key
      }
      this.UserSearchInputError = null
    },
    ShowFormError (str) {
      this.UserSearchInputError = str
    },
    SubmitHost ($event) {
      if ($event) $event.preventDefault()
      if (this.IsUserSearchInputValid !== true)
        this.ShowFormError('Domain must be in host format (examples: abc.xyz, m.example.com)')
      else {
        this.FetchEmojis(this.UserSearchInput)
          .then(emojis => {
            this.loadedEmojis = emojis
          })
          .catch(err => {
            this.ShowFormError(err.message)
          })
      }
    },
    FetchEmojis () {
      this.$nextTick(this.$nuxt.$loading.start)
      return new Promise((resolve, reject) => {
        const host = this.UserSearchInput
        let url = `/api/emoji_proxy/${host}`
        if (this.isDev) url = `https://${host}/api/v1/custom_emojis`
        axios.get(url)
          .then(response => {
            const { data } = response
            this.$nuxt.$loading.finish()
            
            // Validation stuff
            if (!Array.isArray(data)) {
              reject(new Error('Instance did not return valid data, check the host name and try again'))
            }
            if (data.length === 0) {
              reject(new Error('This instance did not respond with any emojis.'))
            }
            if (data[0] == null) {
              reject(new Error('This instance responded with invalid data.'))
            }
            const properties = ['shortcode', 'url']
            properties.forEach(property => {
              if (data[0][property] == null) {
                reject(new Error(`Returned data failed validation, missing <u>${property}</u> property.`))
              }
            })

            let filteredData = data.map(({ shortcode, url }, index) => ({
              url, shortcode: shortcode.toLowerCase(), id: index.toString(16)
            }))

            TimSort.sort(filteredData, (a, b) => {
              const ObjA = a.shortcode
              const ObjB = b.shortcode
              if (ObjA < ObjB) return -1
              else if (ObjA > ObjB) return 1
              else return 0
            })

            let hashtable = {}
            const noDuplicatesData = filteredData.filter(({ shortcode }) => {
              if (hashtable[shortcode] == null) return (hashtable[shortcode] = true)
              return false
            })

            resolve(noDuplicatesData)
          })
          .catch(err => {
            this.$nuxt.$loading.finish()
            reject(err)
          })
      })
    },
    EmojiSearchInputChange () {
      const query = this.EmojiSearchInput
      if (query != null && query.trim() !== '' && this.IsEmojiSearchFilled) {
        this.SearchEmoji(query.trim())
      } else {
        this.EmojiSearchResults = null
      }
    },
    SearchEmoji (query) {
      const FuseInstance = new Fuse(this.loadedEmojis, {
        shouldSort: true,
        threshold: 0.3,
        location: 0,
        distance: 50,
        maxPatternLength: 24,
        minMatchCharLength: 2,
        keys: ['shortcode']
      })
      this.EmojiSearchResults = FuseInstance.search(query)
    },
    IsHostValid (host) {
      return host.match(/^([^/]){1,}\.([^/]){2,}$/g) !== null
    }
  }
}
</script>

<style>
.searchbox {
  border: 0;
  background-color: transparent;
  outline: none !important;
  appearance: none;
  -moz-appearance: none;
  -ms-appearance: none;
  -webkit-appearance: none;
  width: 100%;
  display: block;
  margin: 0 0 1rem 0;
  line-height: 1.5;
}
.at {
  margin-bottom: 1rem;
  display: block;
  line-height: 1.5;
}
.error-box {
  background-color: #e8467a;
  color: #0f1414;
  padding: 0.25rem 0.65rem;
  line-height: 1.3;
}
.emojis-box {
  min-height: 50vh;
  max-height: 50vh;
  overflow-y: auto;
}
.emojis-scroll, .emoji-search-results {
  height: 100%;
  max-height: 100%;
  overflow-y: auto;
}
.emoji-row {
    display: flex;
    flex-direction: row;
    height: 78px;
}

.emoji-row-image-container {
    min-width: 78px;
    max-width: 78px;
    display: flex;
    flex-direction: column;
}

.emoji-row-image {
    display: block;
    width: auto;
    max-width: 100%;
    margin: auto;
    max-height: 80%;
}

.emoji-row-header-container {
    display: flex;
    flex-direction: row;
    height: auto;
}

.emoji-row-name {
  display: block;
  margin: auto 0;
}

.emojis-scroll {
  padding: 0 17pt;
}

.emojis-box {
  border: 2pt solid #e8467a;
}

.emoji-search {
  border: 2pt solid #e8467a;
  border-bottom: 0;
}

.emoji-search .searchbox {
  margin: 0;
  padding: 8px 15px;
}

.emoji-row-name, .emoji-row-header-container {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
</style>

