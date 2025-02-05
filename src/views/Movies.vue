<template>
  <div class="container py-5">
    <MovieList
      :initialMovies="trendingMovies"
      :initialTitle="trendingTitle"
      @after-show-modal="afterShowModal"
      @after-add-to-watch="afterAddToWatch"
      @after-delete-to-watch="afterDeleteToWatch"
    />
    <MovieList
      :initialMovies="onShowMovies"
      :initialTitle="onShowTitle"
      @after-show-modal="afterShowModal"
      @after-add-to-watch="afterAddToWatch"
      @after-delete-to-watch="afterDeleteToWatch"
    />
    <MovieList
      :initialMovies="comingMovies"
      :initialTitle="comingTitle"
      @after-show-modal="afterShowModal"
      @after-add-to-watch="afterAddToWatch"
      @after-delete-to-watch="afterDeleteToWatch"
    />
    <MovieList
      :initialMovies="popularityMovies"
      :initialTitle="popularityTitle"
      @after-show-modal="afterShowModal"
      @after-add-to-watch="afterAddToWatch"
      @after-delete-to-watch="afterDeleteToWatch"
    />
    <!-- modal -->
    <MovieDetailLoadingVue v-if="modalIsLoading" />
    <router-view
      v-if="modalVisibility"
      :initialMovie="movie"
      :initialCollectionData="collectionData"
      @after-close-modal="afterCloseModal"
      @after-add-to-watch="afterAddToWatch"
      @after-delete-to-watch="afterDeleteToWatch"
    />
    <div class="modal-backdrop" v-if="modalVisibility || modalIsLoading"></div>
  </div>
</template>

<script>
import moviesAPI from "../apis/movies";
import MovieList from "../components/MovieList.vue";
import MovieDetailLoadingVue from "../components/MovieDetailLoading.vue";

const STORAGE_KEY = "to-watch-movies";

export default {
  components: {
    MovieList,
    MovieDetailLoadingVue,
  },
  created() {
    this.toWatchMovies = JSON.parse(localStorage.getItem(STORAGE_KEY)) || [];
    this.getToday();
    this.fetchTrending();
    this.fetchOnShow();
    this.fetchComing();
    this.fetchPopularity();
  },
  data() {
    return {
      trendingMovies: [],
      onShowMovies: [],
      comingMovies: [],
      popularityMovies: [],
      trendingTitle: "本週熱門",
      onShowTitle: "現正上映",
      comingTitle: "即將上映",
      popularityTitle: "近期受歡迎",
      today: "",
      modalVisibility: false,
      modalIsLoading: false,
      movie: {
        id: 0,
        backdropPath: "",
        genres: [],
        belongsToCollection: {},
        title: "",
        originalTitle: "",
        releaseDate: "",
        runtime: 0,
        overview: "",
        voteAverage: 0,
        voteCount: 0,
        credits: {},
        inToWatch: false,
      },
      collectionData: [],
      toWatchMovies: [],
    };
  },
  methods: {
    filterToWatch(unfilteredMovies) {
      let filteredToWatchMovies = unfilteredMovies.map((movie) => {
        if (this.toWatchMovies.length > 0) {
          for (let i = 0; i < this.toWatchMovies.length; i++) {
            if (movie.id === this.toWatchMovies[i].id) {
              movie.inToWatch = true;
            }
          }
          return movie;
        } else {
          return movie;
        }
      });

      return filteredToWatchMovies;
    },
    filterDeleteToWatch(movieId, unfilteredMovies) {
      let filteredToWatchMovies = unfilteredMovies.map((unfilteredMovie) => {
        if (unfilteredMovie.id === movieId) {
          return {
            ...unfilteredMovie,
            inToWatch: false,
          };
        } else {
          return unfilteredMovie;
        }
      });

      return filteredToWatchMovies;
    },
    saveStorage() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(this.toWatchMovies));
    },
    getToday() {
      const date = new Date();
      const year = date.getFullYear();
      let month = date.getMonth() + 1;
      let day = date.getDate();

      if (month < 10) {
        month = 0 + month.toString();
      }
      if (day < 10) {
        day = 0 + day.toString();
      }
      this.today = year + "-" + month + "-" + day;
    },
    async fetchTrending() {
      try {
        const { data } = await moviesAPI.getTrending();
        this.trendingMovies = data.results.map((movie) => ({
          id: movie.id,
          posterPath: movie.poster_path,
          title: movie.title,
          originalTitle: movie.original_title,
          releaseDate: movie.release_date,
          inToWatch: false,
        }));
        this.trendingMovies = this.filterToWatch(this.trendingMovies);
      } catch (error) {
        console.log("fetch trending error", error);
      }
    },
    async fetchOnShow() {
      try {
        const { data } = await moviesAPI.getOnShow({ lte: this.today });
        this.onShowMovies = data.results.map((movie) => ({
          id: movie.id,
          posterPath: movie.poster_path,
          title: movie.title,
          originalTitle: movie.original_title,
          releaseDate: movie.release_date,
          inToWatch: false,
        }));
        this.onShowMovies = this.filterToWatch(this.onShowMovies);
      } catch (error) {
        console.log("fetch onShow error", error);
      }
    },
    async fetchComing() {
      try {
        const { data } = await moviesAPI.getComing({ gte: this.today });
        this.comingMovies = data.results.map((movie) => ({
          id: movie.id,
          posterPath: movie.poster_path,
          title: movie.title,
          originalTitle: movie.original_title,
          releaseDate: movie.release_date,
          inToWatch: false,
        }));
        this.comingMovies = this.filterToWatch(this.comingMovies);
      } catch (error) {
        console.log("fetch coming error", error);
      }
    },
    async fetchPopularity() {
      try {
        const { data } = await moviesAPI.getPopularity();
        this.popularityMovies = data.results.map((movie) => ({
          id: movie.id,
          posterPath: movie.poster_path,
          title: movie.title,
          originalTitle: movie.original_title,
          releaseDate: movie.release_date,
          inToWatch: false,
        }));
        this.popularityMovies = this.filterToWatch(this.popularityMovies);
      } catch (error) {
        console.log("fetch popularity error", error);
      }
    },
    async afterShowModal(movieId) {
      try {
        document.body.style.overflow = "hidden";
        this.modalVisibility = false;
        this.modalIsLoading = true;
        const { data } = await moviesAPI.getMovieDetail({ movieId });

        this.movie = {
          ...this.movie,
          id: data.id,
          backdropPath: data.backdrop_path,
          posterPath: data.poster_path,
          genres: data.genres,
          belongsToCollection: data.belongs_to_collection,
          title: data.title,
          originalTitle: data.original_title,
          releaseDate: data.release_date,
          runtime: data.runtime,
          overview: data.overview,
          voteAverage: data.vote_average,
          voteCount: data.vote_count,
          credits: data.credits,
          inToWatch: false,
        };

        if (this.toWatchMovies.length > 0) {
          for (let i = 0; i < this.toWatchMovies.length; i++) {
            if (Number(movieId) === this.toWatchMovies[i].id) {
              this.movie.inToWatch = true;
            }
          }
        }

        if (data.belongs_to_collection) {
          let collectionId = data.belongs_to_collection.id;
          const response = await moviesAPI.getCollection({ collectionId });
          this.collectionData = response.data.parts.filter(
            (collection) => collection.id !== this.movie.id
          );
        } else {
          this.collectionData = null;
        }
        this.modalIsLoading = false;
        this.modalVisibility = true;
      } catch (error) {
        this.modalIsLoading = false;
        console.log("after show modal error", error);
      }
    },
    afterCloseModal() {
      this.modalVisibility = false;
      document.body.removeAttribute("style");
      this.$router.push("/dont-forget-movies");
    },
    afterAddToWatch(movie) {
      movie.inToWatch = true;
      const listMovie = {
        id: movie.id,
        title: movie.title,
        originalTitle: movie.originalTitle,
        posterPath: movie.posterPath,
        releaseDate: movie.releaseDate,
        inToWatch: movie.inToWatch,
      };
      this.toWatchMovies.push(listMovie);
      this.trendingMovies = this.filterToWatch(this.trendingMovies);
      this.onShowMovies = this.filterToWatch(this.onShowMovies);
      this.comingMovies = this.filterToWatch(this.comingMovies);
      this.popularityMovies = this.filterToWatch(this.popularityMovies);
    },
    afterDeleteToWatch(movie) {
      movie.inToWatch = false;
      this.toWatchMovies = this.toWatchMovies.filter(
        (toWatchMovie) => toWatchMovie.id !== movie.id
      );
      this.trendingMovies = this.filterDeleteToWatch(
        movie.id,
        this.trendingMovies
      );
      this.onShowMovies = this.filterDeleteToWatch(movie.id, this.onShowMovies);
      this.comingMovies = this.filterDeleteToWatch(movie.id, this.comingMovies);
      this.popularityMovies = this.filterDeleteToWatch(
        movie.id,
        this.popularityMovies
      );
    },
  },
  watch: {
    toWatchMovies: {
      handler: function () {
        this.saveStorage();
      },
      deep: true,
    },
  },
  beforeRouteUpdate(to, from, next) {
    const { id: movieId } = to.params;

    if (movieId) {
      this.afterShowModal(movieId);
    } else {
      this.modalVisibility = false;
      document.body.removeAttribute("style");
    }

    next();
  },
};
</script>

<style lang="scss" scoped>
.modal-backdrop {
  opacity: 0.5;
}
</style>