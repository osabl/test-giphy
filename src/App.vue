<template>
  <v-app>
    <header>
      <v-container>
        <v-row class="center">
          <v-col cols="3">
            <v-img
              :src="require('@/assets/logo.svg')"
              contain
              height="50"
            />
          </v-col>
          <v-col cols="9">
            <v-text-field
              v-model="query"
              @input="search(20)"
              :placeholder="'Search'"
              :outlined="true"
              :color="'blue'"
              :prepend-inner-icon="'mdi-magnify'"
              :rounded="true"
            ></v-text-field>
          </v-col>
        </v-row>
      </v-container>
    </header>

    <v-main>
      <v-container>
        <v-fade-transition
          group
          class="row"
          v-scroll="infiniteLoad"
          tag="div"
        >
          <v-col cols="6" sm="4" md="3" v-for="(gif, index) in gifs.data" :key="gif.id + index">
            <v-card>
              <v-img
                :src="gif.images.downsized.url"
                :lazy-src="gif.images.preview_gif.url"
                :aspect-ratio="1"
              >
                <template v-slot:placeholder>
                  <v-row
                    class="fill-height"
                    align="center"
                    justify="center"
                  >
                    <v-progress-circular indeterminate color="blue"></v-progress-circular>
                  </v-row>
                </template>
              </v-img>
              <v-card-actions v-if="webShareApiSupported">
                <v-spacer></v-spacer>
                <v-btn
                  icon
                  color="blue"
                  @click="share(gif.url, gif.title)"
                >
                  <v-icon>mdi-share-variant</v-icon>
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-col>
        </v-fade-transition>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>
import debounce from './debounce'

export default {
  data: () => ({
    gifs: {
      source: 'random',
      query: '',
      offset: 0,
      data: []
    },
    API_KEY: 'vKAKk37t5bqwLBVFsyEZwWWOkDxnoDdC',
    query: ''
  }),

  mounted () {
    this.addGifs(20)
  },

  computed: {
    webShareApiSupported () {
      return navigator.share
    }
  },

  methods: {
    async getRandomGif (apiKey, tag = '') {
      try {
        const url = `https://api.giphy.com/v1/gifs/random?api_key=${apiKey}&tag=${tag}`
        const response = await fetch(url)
        const result = await response.json()
        return result.data
      } catch (err) {
        console.error(err)
      }
    },

    async getDesiredGifs (apiKey, query, limit = 20, offset = 0) {
      try {
        const url = `https://api.giphy.com/v1/gifs/search?api_key=${apiKey}&q=${query}&limit=${limit}&offset=${offset}`
        const response = await fetch(url)
        const result = await response.json()
        return result.data
      } catch (err) {
        console.error(err)
      }
    },

    async addRandomGifs (number) {
      this.loading = true
      const gifs = []

      for (let i = 0; i < number; i++) {
        gifs.push(this.getRandomGif(this.API_KEY))
      }

      try {
        const res = await Promise.all(gifs)
        this.gifs.data.push(...res)
      } catch (err) {
        console.error(err)
      }

      this.loading = false
    },

    async addDesiredGifs (query, limit) {
      const gifs = await this.getDesiredGifs(this.API_KEY, query, limit, this.gifs.offset)

      if (gifs.length > 0) {
        this.gifs.offset += limit
        this.gifs.data.push(...gifs)
      } else {
        const notFoundGif = await this.getDesiredGifs(this.API_KEY, 'Not Found', 1, Math.floor(Math.random() * 100))
        this.gifs.data.push(...notFoundGif)
      }
    },

    addGifs (number) {
      if (this.query.trim()) {
        const query = this.query.trim()

        if (this.gifs.source !== 'search' || this.gifs.query !== query) {
          this.gifs.source = 'search'
          this.gifs.query = query
          this.gifs.offset = 0
          this.gifs.data.length = 0
        }

        this.addDesiredGifs(query, number)
      } else {
        if (this.gifs.source !== 'random') {
          this.gifs.source = 'random'
          this.gifs.query = ''
          this.gifs.offset = 0
          this.gifs.data.length = 0
        }

        this.addRandomGifs(number)
      }
    },

    infiniteLoad (event) {
      const target = event.target
      const isEndScroll = target.scrollingElement.scrollHeight - target.scrollingElement.scrollTop <= target.scrollingElement.clientHeight
      if (isEndScroll) {
        this.addGifs(20)
      }
    },

    search: debounce(function (number) {
      this.gifs.data.length = 0
      this.addGifs(number)
    }, 500),

    share (url, text) {
      navigator.share({
        text,
        url
      })
    }
  }
}
</script>
