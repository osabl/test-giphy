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
          id="infinity-scroll"
        >
          <v-col cols="3" v-for="(gif, index) in gifs.data" :key="index">
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
  name: 'App',

  components: {
    HelloWorld
  },

  data: () => ({
    //
  })
}
</script>
