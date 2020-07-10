<template>
  <v-app>
    <header>
      <v-container>
        <v-row class="center">
          <v-col>
            <v-img
              :src="require('@/assets/logo.svg')"
              contain
              height="50"
            />
          </v-col>
          <v-col cols="9">
            <v-text-field
              v-model="query"
              @input="addGifs(20)"
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
        <v-row
          v-scroll="infiniteLoad"
        >
          <v-col cols="3" v-for="(gif, index) in gifs.data" :key="gif.id + index">
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
            </v-card>
          </v-col>
        </v-row>
      </v-container>
    </v-main>
  </v-app>
</template>

<script>

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

    addRandomGifs (number) {
      const gifs = []

      for (let i = 0; i < number; i++) {
        gifs.push(this.getRandomGif(this.API_KEY))
      }

      Promise.all(gifs)
        .then(res => this.gifs.data.push(...res))
        .catch(err => console.error(err))
    },

    async addDesiredGifs (query, limit) {
      const gifs = await this.getDesiredGifs(this.API_KEY, query, limit, this.gifs.offset)

      if (gifs.length > 0) {
        this.gifs.data.push(...gifs)
        this.gifs.offset += limit
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
          this.gifs.data = []
        }

        this.addDesiredGifs(query, number)
      } else {
        if (this.gifs.source !== 'random') {
          this.gifs.source = 'random'
          this.gifs.query = ''
          this.gifs.offset = 0
          this.gifs.data = []
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
    }
  }
}
</script>
