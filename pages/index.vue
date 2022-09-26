<template>
  <v-row justify="center" align="center">
    <v-col cols="12">
      <v-card>
        <v-card-title class="headline">
          Welcome to "AnyEp"
        </v-card-title>
        <v-card-text>
          <p>Search for a TV show and get a random episode to watch.</p>
        </v-card-text>
        <v-col cols="12">
          <v-autocomplete
            v-model="select"
            :loading="loading"
            :items="items"
            item-text="name"
            :search-input.sync="search"
            cache-items
            class="mx-4"
            flat
            hide-no-data
            hide-details
            label="Select a TV series"
            solo-inverted
            return-object
          ></v-autocomplete>
        </v-col>
        <v-card-actions>
          <v-spacer />
          <v-btn
            color="primary"
            block
            @click="returnRandom"
          >
            Give me a random episode
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-col>

    <v-col cols="12">
      <v-card
        v-if="finalResult !== ''"
        class="mx-auto"
        max-width="344"
      >
        <v-card-text>
          <div>Season {{ finalResult.season_number }}, Episode {{ finalResult.episode_number }}</div>
          <p class="text-h4 text--primary">
            {{ finalResult.name }}
          </p>
          <p>{{ finalResult.air_date }}</p>
          <div class="text--primary">
            {{ finalResult.overview }}
          </div>
        </v-card-text>
      </v-card>
    </v-col>
  </v-row>
</template>

<script>
import { API_KEY } from '~/plugins/key.js'
const axios = require('axios')
export default {
  name: 'IndexPage',
  data() {
    return {
      loading: false,
      items: [],
      search: null,
      select: null,
      selectedSeries: '',
      selectedSeriesDetails: '',
      selectedSeriesSeasonEpisodeNum: 0,
      finalResult: ''
    }
  },
  watch: {
    search (val) {
      val && val !== this.select && this.querySelections(val)
    },
  },
  methods: {
    querySelections (v) {
      this.loading = true

      axios.get(`https://api.themoviedb.org/3/search/tv?api_key=${API_KEY}&page=1&query=${v}`).then(response => {
          this.items = response.data.results
          this.loading = false
      })
    },
    async returnRandom() {
      await axios.get(`https://api.themoviedb.org/3/tv/${this.select.id}?api_key=${API_KEY}&language=en-US`).then(response => {
        this.selectedSeriesDetails = response.data
      })
      const numSeasons = this.selectedSeriesDetails.number_of_seasons
      const randomSeason = Math.floor(Math.random() * (numSeasons + 1) + 1)

      await axios
        .get(`https://api.themoviedb.org/3/tv/${this.select.id}/season/${randomSeason}?api_key=${API_KEY}&language=en-US`)
        .then(response => {
          this.selectedSeriesSeasonEpisodeNum = response.data.episodes.length
        })

      const randomEpisode = Math.floor(Math.random() * (this.selectedSeriesSeasonEpisodeNum + 1) + 1)

      await axios
        .get(`https://api.themoviedb.org/3/tv/${this.select.id}/season/${randomSeason}/episode/${randomEpisode}?api_key=${API_KEY}&language=en-US`)
        .then(response => {
          this.finalResult = response.data
          console.log('FINAL RESULT: ', this.finalResult)
        })
    }
  }
}
</script>
