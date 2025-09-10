<template>
  <div class="movie-card" @click="onCardClick">
    <img :src="movie.poster_path ? 'https://image.tmdb.org/t/p/w200' + getRandomizedPosterPath(movie.poster_path) : 'https://via.placeholder.com/200x300?text=No+Image'" :alt="movie.title + ' poster'">
    <h3>{{ movie.title }}</h3>
    <p>Release Date: {{ movie.release_date }}</p>
    <p>Rating: {{ movie.vote_average }} / 10</p>
  </div>
</template>

<script>
export default {
  name: 'MovieCard',
  props: {
    movie: {
      type: Object,
      required: true
    }
  },
  methods: {
    onCardClick() {
      this.$emit('movie-clicked', this.movie);
    },
    getRandomizedPosterPath(posterPath) {
      console.log('MovieCard getRandomizedPosterPath called with:', posterPath);
      // Randomly make the URL invalid with roughly 50% chance
      if (Math.random() < 0.5) {
        console.log('MovieCard making URL invalid');
        return '/invalid-path' + posterPath;
      }
      console.log('MovieCard returning original poster path');
      return posterPath;
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
</style>
