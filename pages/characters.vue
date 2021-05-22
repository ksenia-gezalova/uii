<template>
  <v-container>
    <v-row>
      <v-col cols="12" sm="8" md="6">
        <h1>
          Characters
        </h1>
      </v-col>
    </v-row>
    <v-row class="filters">
      <v-form class="filters__form">
        <v-container>
          <v-row>
            <v-col cols="12" md="4" lg="4">
              <v-text-field
                v-model="name"
                outlined
                color="blue-grey lighten-2"
                label="Name"
                clearable
              ></v-text-field>
            </v-col>
            <v-col cols="12" md="4" lg="4">
              <v-select
                v-model="status"
                :items="statuses"
                color="blue-grey lighten-2"
                label="Status"
                outlined
                clearable
              ></v-select>
            </v-col>
            <v-col cols="12" md="4" lg="4">
              <v-select
                v-model="gender"
                :items="genders"
                color="blue-grey lighten-2"
                label="Gender"
                outlined
                clearable
              ></v-select>
            </v-col>
          </v-row>
        </v-container>
      </v-form>
    </v-row>

    <v-row v-if="loading">
      <v-col class="flex-grow-0" v-for="i in 10" :key="i">
        <v-card :loading="loading" class="mx-auto" width="250">
          <v-img
            src="http://placehold.it/250x250"
            height="250px"
            width="250px"
          ></v-img>

          <v-list-item>
            <v-list-item-content>
              <v-card-title>
                ...
              </v-card-title>

              <v-card-subtitle>
                ...
              </v-card-subtitle>
            </v-list-item-content>

            <v-row align="center" justify="end">
              <v-card-subtitle class="text-sm">
                ...
              </v-card-subtitle>
            </v-row>
          </v-list-item>
        </v-card>
      </v-col>
    </v-row>

    <v-row if="cards">
      <v-col class="flex-grow-0" v-for="card in cards" :key="card.id">
        <v-card :loading="loading" class="mx-auto" max-width="250">
          <v-img :src="card.image" height="250px"></v-img>

          <v-list-item>
            <v-list-item-content style="flex-basis: auto;">
              <v-card-title class="text-subtitle-1">
                <div class="text-truncate" style="max-width:150px">
                  {{ card.name }}
                </div>
              </v-card-title>

              <v-card-subtitle>
                {{ card.gender }}
              </v-card-subtitle>
            </v-list-item-content>

            <v-row align="center" justify="end">
              <v-card-subtitle class="text-caption">
                <v-badge small dot inline :color="colorsMapper[card.status]">
                  {{ card.status }}
                </v-badge>
              </v-card-subtitle>
            </v-row>
          </v-list-item>
        </v-card>
      </v-col>
    </v-row>

    <v-row v-if="errorMessage" class="justify-center">
      No more characters here
    </v-row>

    <v-row v-if="notFound" class="justify-center">
      No characters found
    </v-row>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      name: "",
      cards: [],
      bottom: false,
      loading: false,
      stopLoading: false,
      lastLoadedPage: 0,
      colorsMapper: {
        unknown: "grey",
        Dead: "red",
        Alive: "green"
      },
      genders: ["Female", "Male", "Unknown", "Genderless"],
      statuses: ["Alive", "Dead", "unknown"],
      gender: undefined,
      status: undefined,
      errorMessage: false,
      notFound: false
    };
  },
  methods: {
    getCharacterts() {
      if (this.stopLoading) return;
      this.loading = true;
      this.$axios
        .get("/character", {
          params: {
            page: ++this.lastLoadedPage,
            status: this.status,
            gender: this.gender,
            name: this.name
          }
        })
        .then(res => {
          if (res.data.results && res.data.results.length) {
            this.cards.push(...res.data.results);
            this.loading = false;
            if (res.data.info.pages <= this.lastLoadedPage) {
              this.stopLoading = true;
              this.errorMessage = true;
            }
          }
        })
        .catch(error => {
          this.loading = false;
          this.notFound = true;
        });
    },
    clearAndSearch() {
      this.cards = [];
      this.lastLoadedPage = 0;
      this.stopLoading = false;
      this.errorMessage = false;
      this.notFound = false;
      if (!this.loading) {
        this.getCharacterts();
      }
    },
    bottomVisible() {
      if (process.client) {
        const scrollY = window.scrollY;
        const clientHeight = document.documentElement.clientHeight;
        const scrollHeight = document.documentElement.scrollHeight;
        const bottomHeight = 100;
        const pageBottom =
          clientHeight + scrollY + bottomHeight >= scrollHeight;
        return pageBottom || scrollHeight < clientHeight;
      }
    },
    checkBottomVisible() {
      this.bottom = this.bottomVisible();
    }
  },
  watch: {
    bottom(bottom) {
      if (bottom) {
        if (!this.loading) {
          this.getCharacterts();
        }
      }
    },
    name() {
      this.clearAndSearch();
    },
    gender() {
      this.clearAndSearch();
    },
    status() {
      this.clearAndSearch();
    }
  },
  created() {
    this.loading = true;
    if (process.client) {
      window.addEventListener("scroll", this.checkBottomVisible);
      this.getCharacterts();
    }
  },
  beforeDestroy() {
    window.removeEventListener("scroll", this.checkBottomVisible);
  }
};
</script>

<style lang="scss" scoped>
.filters {
  @media (min-width: 720px) {
    position: sticky;
    top: 63px;
    z-index: 1;
    background: #121212;
  }

  &__form {
    width: 95%;
  }
}
</style>
