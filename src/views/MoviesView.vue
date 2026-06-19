<script setup>
import { onMounted, ref, computed } from 'vue'
import { useMovieStore } from '../stores/movieStore'

const store = useMovieStore()

// 현재 선택된 정렬 기준 ('title' | 'release_date' | 'rating')
const sortType = ref('title')
const currentPage = ref(1)
const moviesPerPage = 8

// 원본 배열을 건드리지 않고 복사본을 정렬해 화면에 반영
const sortedMovies = computed(() => {
  const list = [...store.movies]

  if (sortType.value === 'title') {
    return list.sort((a, b) => a.title.localeCompare(b.title, 'ko'))
  }

  if (sortType.value === 'release_date') {
    return list.sort((a, b) => {
      if (!a.release_date) return 1
      if (!b.release_date) return -1
      return b.release_date.localeCompare(a.release_date)
    })
  }

  if (sortType.value === 'rating') {
    return list.sort((a, b) => b.vote_average - a.vote_average)
  }

  return list
})

const totalPages = computed(() => {
  return Math.ceil(sortedMovies.value.length / moviesPerPage)
})

const pageNumbers = computed(() => {
  const numbers = []

  for (let page = 1; page <= totalPages.value; page++) {
    numbers.push(page)
  }

  return numbers
})

const paginatedMovies = computed(() => {
  const startIndex = (currentPage.value - 1) * moviesPerPage
  const endIndex = startIndex + moviesPerPage

  return sortedMovies.value.slice(startIndex, endIndex)
})

const changePage = (page) => {
  currentPage.value = page
}

const changeSortType = (type) => {
  sortType.value = type
  currentPage.value = 1
}

onMounted(() => {
  store.fetchMovies()
  // [12주차 추가] 상세 정보에 갔다 돌아왔을 때 브라우저 탭 이름을 원래대로 복구합니다.
  document.title = '🍿 국내 극장 화제작 (인기순)'
})
</script>

<template>
  <main class="page">
    <div class="header-section">
      <h1>🍿 국내 극장 화제작 (인기순)</h1>
      <p class="sub-title"> 2025년 이후 국내 정식 개봉한 실시간 인기 상영작</p>

      <div class="sort-controls">
        <span class="sort-label">정렬 기준</span>
        <button
          type="button"
          class="sort-btn"
          :class="{ active: sortType === 'title' }"
          @click="changeSortType('title')"
        >
          제목순
        </button>
        <button
          type="button"
          class="sort-btn"
          :class="{ active: sortType === 'release_date' }"
          @click="changeSortType('release_date')"
        >
          개봉일순
        </button>
        <button
          type="button"
          class="sort-btn"
          :class="{ active: sortType === 'rating' }"
          @click="changeSortType('rating')"
        >
          평점순
        </button>
      </div>
    </div>

    <div v-if="store.isLoading" class="status-message loading">
      ⌛ 실시간 국내 개봉작 데이터를 싣고 오는 중입니다...
    </div>

    <div v-else-if="store.errorMessage" class="status-message error">
      🚨 {{ store.errorMessage }}
    </div>

    <div v-else>
      <div class="movie-list">
        <div v-for="movie in paginatedMovies" :key="movie.id" class="movie-card">
          <img
            v-if="movie.poster_path"
            :src="`https://image.tmdb.org/t/p/w500${movie.poster_path}`"
            :alt="movie.title"
            class="poster"
          />
          <div v-else class="poster-placeholder">이미지 준비 중</div>
          <div class="card-content">
            <!-- 11주차 내용 동일 -->
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
            <button
              @click="store.toggleFavorite(movie.id)"
              :class="{ active: movie.isFavorite }"
              class="fav-btn"
            >
              {{ movie.isFavorite ? '❤️ 찜 해제' : '🤍 찜하기' }}
            </button>
            <!-- [12주차 RouterLink 추가] -->
            <RouterLink
              :to="`/movies/${movie.id}`"
              class="stretched-link"
              :aria-label="`${movie.title} 상세 정보 보기`"
            />
          </div>
        </div>
      </div>

      <div v-if="totalPages > 1" class="pagination">
        <button
          v-for="page in pageNumbers"
          :key="page"
          type="button"
          class="page-btn"
          :class="{ active: currentPage === page }"
          @click="changePage(page)"
        >
          {{ page }}
        </button>
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

.sort-controls {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 10px;
  margin-top: 20px;
}

.sort-label {
  font-size: 14px;
  font-weight: bold;
  color: #34495e;
}

.sort-btn {
  padding: 8px 16px;
  border: 1px solid #ddd;
  border-radius: 20px;
  background: white;
  color: #555;
  font-weight: bold;
  font-size: 14px;
  cursor: pointer;
  transition: 0.2s;
}

.sort-btn:hover {
  border-color: #42b883;
  color: #42b883;
}

.sort-btn.active {
  background: #42b883;
  border-color: #42b883;
  color: white;
}

.status-message {
  text-align: center;
  font-size: 20px;
  font-weight: bold;
  padding: 50px;
  border-radius: 12px;
}

.loading {
  color: #3498db;
  background-color: #e3f2fd;
}

.error {
  color: #e74c3c;
  background-color: #fdeaea;
}

.movie-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 30px;
}

.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
  gap: 8px;
  margin-top: 35px;
}

.page-btn {
  width: 38px;
  height: 38px;
  border: 1px solid #ddd;
  border-radius: 50%;
  background: white;
  color: #555;
  font-weight: bold;
  cursor: pointer;
  transition: 0.2s;
}

.page-btn:hover {
  border-color: #ff4757;
  color: #ff4757;
}

.page-btn.active {
  background: #ff4757;
  border-color: #ff4757;
  color: white;
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

.fav-btn {
  position: relative;
  z-index: 2;
  width: 100%;
  padding: 12px;
  cursor: pointer;
  border: none;
  background: #ecf0f1;
  color: #333;
  border-radius: 8px;
  font-weight: bold;
  font-size: 14px;
  transition: 0.3s;
  margin-top: auto;
}

.fav-btn.active {
  background: #ff4757;
  color: white;
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
