<template>
  <v-app>
    <v-container>
      <!-- Alert -->
      <v-alert text v-model="alert.show" type="error" dismissible>{{
        alert.message
      }}</v-alert>
      <!-- Title -->
      <v-row justify="center" class="mt-5">
        <h1 class="font-weight-light">Lyrics Finder</h1>
      </v-row>
      <!-- Form -->
      <v-row justify="center">
        <v-col md="5" sm="12">
          <v-card class="ma-3">
            <v-card-text>
              <v-form
                class="ma-3"
                @submit.prevent="searchSong()"
                ref="searchForm"
              >
                <v-text-field
                  label="Artist"
                  prepend-icon="mic"
                  :rules="artistRules"
                  v-model="search.artist"
                  autocomplete="off"
                ></v-text-field>
                <v-text-field
                  label="Song"
                  prepend-icon="album"
                  :rules="songRules"
                  v-model="search.song"
                  autocomplete="off"
                ></v-text-field>
                <v-row justify="center">
                  <v-btn class="primary mx-3" type="submit">SEARCH</v-btn>
                  <v-btn @click="translateSong()" class="secondary mx-3">TRANSLATE</v-btn>
                </v-row>
                <v-textarea
                  filled
                  label="lyrics"
                  v-model="lyrics"
                  class="mt-3"
                ></v-textarea>
                <p>Song has {{stats.words}} words and {{stats.lines}} lines</p>
                <v-textarea
                  filled
                  label="lyrics"
                  v-model="stats.translate"
                  class="mt-3"
                ></v-textarea>
              </v-form>
            </v-card-text>
          </v-card>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import axios from 'axios';

export default {
  name: "App",
  data: () => ({
    search: { artist: "", song: "" },
    lyrics: "",
    alert: { show: false, message: "" },
    stats: {words: 0, lines: 0, translate: ""},
    artistRules: [(value) => !!value || "Artist is required"],
    songRules: [(value) => !!value || "Song is required"],
  }),
  methods: {
    async searchSong() {
      let valid = this.$refs.searchForm.validate(); //Validate form
      if (valid) {
        try {
          this.lyrics = "";
          const res = await this.axios.get(
            "/" + this.search.artist + "/" + this.search.song
          );
          if (res.data.lyrics.length > 0) {
            //If there is lyrics
            this.lyrics = res.data.lyrics;
          } else {
            this.alert = { show: true, message: "Lyrics not found" };
          }
        } catch (error) {
          console.log(error);
        }
      }
    },
    async translateSong() {
      try {
        const res = await axios.post("http://localhost:3000/stats", {song: this.lyrics});
        console.log(res);
        const trans = await axios.post("http://localhost:3000/translate", {song: this.lyrics});
        this.stats = {words: res.data.words, lines: res.data.lines, translate: trans.data.result};
      } catch (error) {
        console.log(error);
      }
    },
  },
};
</script>
