<template>
  <div class="movie-card" @click="onCardClick">
    <img :src="movie.poster_path ? 'https://image.tmdb.org/t/p/w200' + movie.poster_path : 'https://via.placeholder.com/200x300?text=No+Image'" :alt="movie.title + ' poster'">
    <h3>{{ movie.title }}</h3>
    <p>Release Date: {{ movie.release_date }}</p>
    <p>Rating: {{ movie.vote_average }} / 10</p>
    <button @click.stop="toggleSave" class="save-button">
      {{ isSaved ? 'Unsave' : 'Save for Later' }}
    </button>
  </div>
</template>

<script>
export default {
  name: 'MovieCard',
  props: {
    movie: {
      type: Object,
      required: true
    },
    isSaved: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    onCardClick() {
      this.$emit('movie-clicked', this.movie);
    },
    toggleSave() {
      this.$emit('toggle-save-movie', this.movie);
    }
  }
}
</script>

<style scoped>
.movie-card {
  border: 1px solid #ccc;
  padding: 16px;
  margin: 16px;
  width: 220px;
  text-align: center;
  cursor: pointer;
}
.movie-card img {
  max-width: 100%;
  height: auto;
  margin-bottom: 10px;
}
.movie-card h3 {
  white-space: nowrap; 
  overflow: hidden;    

  width: 100%; 
}

.save-button {
  background-color: #007bff;
  color: white;
  border: none;
  padding: 8px 12px;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.save-button:hover {
  background-color: #0056b3;
}
</style>