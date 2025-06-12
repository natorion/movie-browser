<template>
  <div id="app">
    <header>
      <h1>Movie Browser</h1>
    </header>
    <main>
      <div class="controls-wrapper">
        <button @click="showWatchLaterList = !showWatchLaterList" class="toggle-watch-later-button">
          {{ showWatchLaterList ? 'Show All Movies' : 'Show Watch Later' }}
        </button>
        <div class="filters-container" v-if="!showWatchLaterList">
          <div class="filter-group">
            <label for="sort-by">Sort By:</label>
            <select id="sort-by" v-model="filters.sortBy">
              <option value="popularity.desc">Popularity (Desc)</option>
              <option value="popularity.asc">Popularity (Asc)</option>
              <option value="release_date.desc">Release Date (Desc)</option>
              <option value="release_date.asc">Release Date (Asc)</option>
              <option value="vote_average.desc">Rating (Desc)</option>
              <option value="vote_average.asc">Rating (Asc)</option>
            </select>
          </div>
          <div class="filter-group">
            <label for="min-rating">Min Rating (0-10):</label>
            <input type="number" id="min-rating" v-model.number="filters.minRating" min="0" max="10" step="0.1">
          </div>
          <button @click="applyFilters" class="apply-filters-button">Apply Filters</button>
        </div>
      </div>

      <div v-if="isLoading" class="loading-message">Loading movies...</div>
      <div v-if="error" class="error-message">{{ error }}</div>
      <div v-if="!isLoading && !error && displayedMovies.length === 0 && !showWatchLaterList" class="no-movies-message">
        No movies found. Try adjusting your filters or check back later.
      </div>
      <div v-if="!isLoading && !error && displayedMovies.length === 0 && showWatchLaterList" class="no-movies-message">
        Your watch later list is empty. Add some movies!
      </div>
      <div v-if="!isLoading && !error && displayedMovies.length > 0" class="movie-list-container">
        <MovieCard 
          v-for="movie in displayedMovies"
          :key="movie.id" 
          :movie="movie"
           :isSaved="watchLaterMovies.some(savedMovie => savedMovie.id === movie.id)"
           @movie-clicked="openDetailModal"
           @toggle-save-movie="handleToggleSaveMovie"
        />
      </div>
      <!-- Router view would go here if you're using Vue Router for different pages -->
      <!-- <router-view/> -->
       <!-- Movie Detail Modal -->
      <div v-if="showDetailModal && selectedMovie" class="modal-overlay" @click.self="closeDetailModal">
        <div class="modal-content">
          <button class="modal-close-button" @click="closeDetailModal">&times;</button>
          <h2>{{ selectedMovie.title }}</h2>
          <img :src="selectedMovie.poster_path ? 'https://image.tmdb.org/t/p/w300' + selectedMovie.poster_path : 'https://via.placeholder.com/300x450?text=No+Image'" :alt="selectedMovie.title + ' poster'" class="modal-poster" v-if="selectedMovie.poster_path">
          <p><strong>Release Date:</strong> {{ selectedMovie.release_date }}</p>
          <p><strong>Rating:</strong> {{ selectedMovie.vote_average }} / 10 ({{ selectedMovie.vote_count }} votes)</p>
          <p><strong>Overview:</strong></p>
          <p class="movie-overview">{{ selectedMovie.overview || 'No overview available.' }}</p>
          <button
            v-if="selectedMovie && watchLaterMovies.some(savedMovie => savedMovie.id === selectedMovie.id)"
            @click="handleToggleSaveMovie(selectedMovie)"
            class="modal-remove-button">
            Remove from Watch Later
          </button>
        </div>
      </div>
    </main>
  </div>
</template>

<script>
import MovieCard from './components/MovieCard.vue';

export default {
  name: 'App',
  components: {
    MovieCard
  },
  data() {
    return {
      movies: [], // This will hold our movie data
      isLoading: true, // Start with loading true as we fetch on created
      error: null,
      filters: {
        sortBy: 'popularity.desc', // Default sort order
        minRating: 0, // Default minimum rating
      },
      selectedMovie: null,
      showDetailModal: false,
      apiKey: process.env.VUE_APP_TMDB_API_KEY,
      watchLaterMovies: [], // Added for watch later list
      showWatchLaterList: false // Added to toggle watch later list display
    };
  },
  computed: {
    displayedMovies() {
      if (this.showWatchLaterList) {
        return this.watchLaterMovies;
      }
      return this.movies;
    }
  },
  async created() {
    this.loadWatchLaterList(); // Load watch later list
    await this.fetchMovies();
  },
  methods: {
    loadWatchLaterList() {
      const list = localStorage.getItem('watchLaterMovies');
      if (list) {
        this.watchLaterMovies = JSON.parse(list);
      }
    },
    saveWatchLaterList() {
      localStorage.setItem('watchLaterMovies', JSON.stringify(this.watchLaterMovies));
    },
    handleToggleSaveMovie(movie) {
      const index = this.watchLaterMovies.findIndex(m => m.id === movie.id);
      if (index === -1) {
        this.watchLaterMovies.push(movie);
      } else {
        this.watchLaterMovies.splice(index, 1);
      }
      this.saveWatchLaterList();
    },
    async fetchMovies() {
      this.isLoading = true;
      this.error = null;
      this.movies = []; // Clear previous movies

      let url = `https://api.themoviedb.org/3/discover/movie?api_key=${this.apiKey}&language=en-US&page=1`;

      // Add filter parameters
      if (this.filters.sortBy) {
        url += `&sort_by=${this.filters.sortBy}`;
      }
      if (this.filters.minRating > 0) {
        url += `&vote_average.gte=${this.filters.minRating}`;
      }
      // You can also add vote_count.gte to ensure movies have a minimum number of votes
      // url += `&vote_count.gte=100`; 

      try {
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error(`HTTP error! status: ${response.status}`);
        }
        const data = await response.json();
        if (data && data.results) {
          this.movies = data.results;
        } else {
          console.warn("No results found or unexpected API response format:", data);
        }
      } catch (e) {
        console.error("Error fetching movies:", e);
        this.error = "Failed to load movies. Please try again later.";
      } finally {
        this.isLoading = false;
      }
    },
    applyFilters() {
      this.fetchMovies();
    },
    openDetailModal(movie) {
      this.selectedMovie = movie;
      this.showDetailModal = true;
    },
    closeDetailModal() {
      this.showDetailModal = false;
      // Optionally clear selectedMovie after a delay for transition, or immediately
      // this.selectedMovie = null; 
    }
  } // methods object closes here
}; // export default closes here
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

header {
  background-color: #42b983;
  color: white;
  padding: 20px;
  margin-bottom: 20px;
}

.controls-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 15px; /* Space between button and filters container */
  margin-bottom: 20px;
}

.filters-container {
  padding: 15px;
  background-color: #f9f9f9;
  border-radius: 8px;
  display: flex;
  flex-wrap: nowrap;
  gap: 20px;
  justify-content: flex-start;
  align-items: center;
  max-width: 450px; /* Or adjust as needed */
  overflow-x: auto;
  /* margin-left and margin-right auto removed as parent .controls-wrapper handles centering */
}

.filter-group {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
}

.filter-group label {
  margin-bottom: 5px;
  font-weight: bold;
  font-size: 0.9em;
}

.filter-group select,
.filter-group input[type="number"] {
  padding: 8px;
  border-radius: 4px;
  border: 1px solid #ccc;
}

.apply-filters-button {
  padding: 10px 20px;
  background-color: #42b983;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1em;
  align-self: flex-end; /* Aligns button with bottom of other inputs if they wrap */
}

.apply-filters-button:hover {
  background-color: #36a471;
}

.toggle-watch-later-button {
  padding: 10px 15px;
  background-color: #007bff;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 1em;
  align-self: center; /* Center button if filters are not shown */
  /* margin-left removed, .controls-wrapper can handle spacing if needed or button takes full width potential */
}

.toggle-watch-later-button:hover {
  background-color: #0056b3;
}

.movie-list-container {
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  gap: 10px; /* Optional: adds space between cards */
}

.loading-message,
.error-message,
.no-movies-message {
  margin-top: 20px;
  font-size: 1.2em;
}

.error-message {
  color: red;
  font-weight: bold;
}


.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  background-color: white;
  padding: 30px;
  border-radius: 8px;
  width: 90%;
  max-width: 600px;
  max-height: 250px;
  overflow-y: hidden; 
  position: relative;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2);
}

.modal-close-button {
  position: absolute;
  top: 15px;
  right: 15px;
  background: none;
  border: none;
  font-size: 1.8em;
  cursor: pointer;
  color: #333;
}

.modal-poster {
  max-width: 200px;
  height: auto;
  float: left;
  margin-right: 20px;
  margin-bottom: 10px;
  border-radius: 4px;
}

.modal-content h2 {
  margin-top: 0;
  color: #2c3e50;
}

.modal-content p {
  line-height: 1.6;
  color: #333;
}

.movie-overview {
  text-align: left;
  margin-bottom: 15px; /* Add some space before the new button */
}

.modal-remove-button {
  background-color: #dc3545; /* Red color for removal */
  color: white;
  border: none;
  padding: 10px 15px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 0.9em;
  display: block; /* Make it a block to take its own line or for margin auto */
  margin-top: 10px; /* Space from the overview */
  margin-left: auto; /* Optional: if you want to push it to the right */
  margin-right: auto; /* Optional: if you want to center it */
}

.modal-remove-button:hover {
  background-color: #c82333; /* Darker red on hover */
}
</style>
