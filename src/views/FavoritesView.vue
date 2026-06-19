<script setup>
import { onMounted } from 'vue'
import { RouterLink } from 'vue-router'
import { useMovieStore } from '../stores/movieStore'

const store = useMovieStore()

onMounted(() => {
  document.title = '❤️ 찜한 작품 목록'
})
</script>

<template>
  <main class="page">
    <div class="header-section">
      <h1>❤️ 찜한 작품 목록</h1>
      <p class="sub-title">찜하기 버튼을 누른 영화만 모아 보는 페이지입니다.</p>
    </div>

    <div v-if="store.favorites.length === 0" class="status-message empty">
      아직 찜한 작품이 없습니다. 영화 목록에서 마음에 드는 작품을 찜해 보세요.
      <RouterLink to="/movies" class="movies-link">영화 목록으로 이동</RouterLink>
    </div>

    <div v-else class="movie-list">
      <div v-for="movie in store.favorites" :key="movie.id" class="movie-card">
        <img
          v-if="movie.poster_path"
          :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
          :alt="movie.title"
          class="poster"
        />
        <div v-else class="poster-placeholder">이미지 준비 중</div>

        <div class="card-content">
          <h3 class="title">{{ movie.title }}</h3>
          <p class="release-date" v-if="movie.release_date">📅 개봉일: {{ movie.release_date }}</p>
          <p class="rating">⭐ {{ movie.vote_average.toFixed(1) }} / 10</p>
          <p class="overview">
            {{
              movie.overview
                ? movie.overview.substring(0, 60) + '...'
                : '국내에 등록된 줄거리 요약 정보가 없습니다.'
            }}
          </p>

          <button @click="store.removeFavorite(movie.id)" class="remove-btn">
            ❤️ 찜 해제
          </button>

          <RouterLink
            :to="`/movies/${movie.id}`"
            class="stretched-link"
            :aria-label="`${movie.title} 상세 정보 보기`"
          />
        </div>
      </div>
    </div>
  </main>
</template>

<style scoped>
.page {
  padding: 40px;
  background-color: #f8f9fa;
  min-height: 100vh;
}

.header-section {
  margin-bottom: 30px;
  text-align: center;
  color: #2c3e50;
}

.sub-title {
  font-size: 14px;
  color: #7f8c8d;
  margin-top: 5px;
}

.status-message {
  text-align: center;
  font-size: 18px;
  font-weight: bold;
  padding: 50px;
  border-radius: 12px;
}

.empty {
  color: #7f8c8d;
  background-color: #ffffff;
}

.movies-link {
  display: block;
  width: fit-content;
  margin: 25px auto 0;
  padding: 12px 18px;
  border-radius: 8px;
  background-color: #42b883;
  color: #ffffff;
  text-decoration: none;
}

.movie-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 30px;
}

.movie-card {
  position: relative;
  border-radius: 12px;
  overflow: hidden;
  background: white;
  text-align: left;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.05);
  transition: transform 0.2s ease;
  display: flex;
  flex-direction: column;
}

.movie-card:hover {
  transform: translateY(-5px);
}

.poster {
  width: 100%;
  height: 380px;
  object-fit: cover;
}

.poster-placeholder {
  width: 100%;
  height: 380px;
  background-color: #ddd;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #7f8c8d;
  font-weight: bold;
}

.card-content {
  padding: 20px;
  display: flex;
  flex-direction: column;
  flex-grow: 1;
}

.title {
  font-size: 18px;
  color: #333;
  margin: 0 0 6px 0;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  font-weight: bold;
}

.release-date {
  font-size: 13px;
  color: #7f8c8d;
  margin-bottom: 10px;
  font-weight: 500;
}

.rating {
  font-weight: bold;
  color: #f39c12;
  margin-bottom: 10px;
  font-size: 16px;
}

.overview {
  font-size: 13px;
  color: #555;
  line-height: 1.4;
  margin-bottom: 20px;
  flex-grow: 1;
}

.remove-btn {
  position: relative;
  z-index: 2;
  width: 100%;
  padding: 12px;
  cursor: pointer;
  border: none;
  background: #ff4757;
  color: white;
  border-radius: 8px;
  font-weight: bold;
  font-size: 14px;
  transition: 0.3s;
  margin-top: auto;
}

.remove-btn:hover {
  background: #ff6b81;
}

.stretched-link {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  z-index: 1;
}
</style>
