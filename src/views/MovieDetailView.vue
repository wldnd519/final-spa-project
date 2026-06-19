<script setup>
import { onMounted, computed, watch, ref } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useMovieStore } from '../stores/movieStore'
import axios from 'axios'

const route = useRoute() // useRoute 주소창 파라미터 추출용
const router = useRouter() // useRouter 뒤로가기 조종용
const store = useMovieStore()

onMounted(() => {
  const movieId = route.params.id // 주소창에서 ID 추출
  store.fetchMovieDetail(movieId) // 스토어 단일 API 호출
})

const formattedBudget = computed(() => {
  const budget = store.selectedMovie?.budget
  return budget && budget !== 0 ? `$${budget.toLocaleString('en-US')}` : '공개되지 않음'
})

const formattedRevenue = computed(() => {
  const revenue = store.selectedMovie?.revenue
  return revenue && revenue !== 0 ? `$${revenue.toLocaleString('en-US')}` : '집계되지 않음'
})

watch(
  () => store.selectedMovie,
  (newMovie) => {
    if (newMovie && newMovie.title) {
      document.title = `${newMovie.title} | NETVUE 상세정보`
    }
  },
  { immediate: true }
)

const goBack = () => {
  router.back() // 브라우저 히스토리 스택을 되돌려 목록 스크롤 위치 보존
}

const aiReview = ref('')
const isAiLoading = ref(false)

const generateAIReview = async () => {
  if (!store.selectedMovie) return
  isAiLoading.value = true
  aiReview.value = ''

  try {
    const promptMessage = `
      너는 영화 평론 유튜버야. 아래 영화 데이터를 기반으로 블로그 글처럼
      2~3문단 분량의 상세하고 흡입력 있는 추천평을 작성해줘(이모지 필수).
      제목: "${store.selectedMovie.title}",
      장르: ${store.selectedMovie.genres.map((g) => g.name).join(',')},
      평점: ${store.selectedMovie.vote_average.toFixed(1)}점
    `

    const response = await axios.post(
      'https://api.groq.com/openai/v1/chat/completions',
      {
        model: 'llama-3.1-8b-instant',
        messages: [{ role: 'user', content: promptMessage }],
      },
      {
        headers: {
          'Content-Type': 'application/json',
          Authorization: `Bearer ${import.meta.env.VITE_AI_API_KEY}`,
        },
      }
    )

    aiReview.value = response.data.choices[0].message.content
  } catch (error) {
    console.error('AI 호출 에러:', error)
    aiReview.value = '서버가 혼잡합니다. 잠시 후 다시 시도해 주세요. 🧐'
  } finally {
    isAiLoading.value = false
  }
}
</script>

<template>
  <main v-if="store.selectedMovie" class="detail-page">
    <div
      class="backdrop-layer"
      :style="{
        backgroundImage: `url(https://image.tmdb.org/t/p/original${store.selectedMovie.backdrop_path})`,
      }"
    >
      <div class="black-curtain"></div>
    </div>
    <div class="content-container">
      <button @click="goBack" class="back-floating-btn">↩ 영화 목록으로 돌아가기</button>
      <div class="movie-hero-grid">
        <div class="poster-zone">
          <img
            v-if="store.selectedMovie?.poster_path"
            :src="`https://image.tmdb.org/t/p/w500${store.selectedMovie.poster_path}`"
            :alt="store.selectedMovie?.title"
            class="main-poster"
          />
          <div v-else class="poster-placeholder">포스터 이미지 없음</div>
        </div>
        <div class="info-zone">
          <h1 class="movie-main-title">{{ store.selectedMovie?.title }}</h1>
          <p class="tagline" v-if="store.selectedMovie?.tagline">
            "{{ store.selectedMovie.tagline }}"
          </p>
          <div class="meta-dashboard">
            <span class="badge rating">⭐ {{ store.selectedMovie?.vote_average.toFixed(1) }} / 10</span>
            <span class="badge runtime">🕒 {{ store.selectedMovie?.runtime }}분</span>
            <span class="badge release">📅 {{ store.selectedMovie?.release_date }} 개봉</span>
          </div>
          <div class="genres-wrapper">
            <span
              v-for="genre in store.selectedMovie?.genres"
              :key="genre.id"
              class="genre-tag"
            >
              {{ genre.name }}
            </span>
          </div>
          <div class="financial-box">
            <div class="financial-item">
              <span class="f-label">총 제작비</span>
              <span class="f-value budget-color">{{ formattedBudget }}</span>
            </div>
            <div class="financial-item">
              <span class="f-label">글로벌 흥행 수익</span>
              <span class="f-value revenue-color">{{ formattedRevenue }}</span>
            </div>
          </div>

          <div class="synopsis-container">
            <h3 class="synopsis-title">시놉시스 줄거리</h3>
            <p class="synopsis-text">
              {{ store.selectedMovie?.overview || '줄거리가 없습니다.' }}
            </p>
          </div>

          <hr class="divider" />

          <div class="ai-section">
            <button @click="generateAIReview" class="ai-btn" :disabled="isAiLoading">
              {{ isAiLoading ? '🤖 생성 중...' : '✨ AI 맞춤 추천사 듣기' }}
            </button>

            <div v-if="aiReview" class="ai-result-box">
              <p>{{ aiReview }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </main>

  <div v-else-if="store.isLoading" class="full-screen-loading-gate">
    <div class="loading-spinner"></div>
    <p class="loading-text">시네마틱 데이터 센터로부터 초고화질 상세 정보를 퍼 올리는 중입니다...</p>
  </div>

  <div v-else-if="store.errorMessage" class="full-screen-error-gate">
    <span class="error-icon">🚨</span>
    <h2 class="error-title">시스템 경고가 발생했습니다</h2>
    <p class="error-msg">{{ store.errorMessage }}</p>
    <button @click="router.push('/movies')" class="error-return-btn">
      안전한 영화 목록 페이지로 도망치기
    </button>
  </div>
</template>

<style scoped>
/* 페이지 전체 다크 모드 및 배경 스틸컷 연출 */
.detail-page {
  position: relative;
  min-height: 100vh;
  color: #ffffff;
  background-color: #0c1014;
  overflow-x: hidden;
}

.backdrop-layer {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-size: cover;
  background-position: center top;
  z-index: 0;
  transform: scale(1.05);
  filter: blur(4px);
}

.black-curtain {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background:
    linear-gradient(to right, #0c1014 25%, rgba(12, 16, 20, 0.8) 60%, #0c1014 100%),
    linear-gradient(to bottom, transparent 50%, #0c1014 100%);
}

/* 중앙 정렬 콘텐츠 컨테이너 및 그리드 배치 */
.content-container {
  position: relative;
  z-index: 1;
  max-width: 1200px;
  margin: 0 auto;
  padding: 50px 30px;
}

.back-floating-btn {
  background: rgba(255, 255, 255, 0.15);
  border: 1px solid rgba(255, 255, 255, 0.2);
  color: #ffffff;
  padding: 12px 24px;
  border-radius: 40px;
  cursor: pointer;
  font-weight: 700;
  font-size: 15px;
  margin-bottom: 40px;
  transition: all 0.3s ease;
  box-shadow: 0 4px 15px rgba(0, 0, 0, 0.3);
}

.back-floating-btn:hover {
  background: #ff4757;
  color: #fff;
  transform: translateY(-3px);
  border-color: #ff4757;
}

.movie-hero-grid {
  display: grid;
  grid-template-columns: 350px 1fr;
  gap: 60px;
  align-items: start;
  margin-top: 20px;
}

/* 타이틀, 포스터, 배지 등 미시 컴포넌트 CSS */
.main-poster {
  width: 100%;
  border-radius: 20px;
  box-shadow: 0 15px 40px rgba(0, 0, 0, 0.7);
  border: 1px solid rgba(255, 255, 255, 0.1);
}

.poster-placeholder {
  width: 100%;
  height: 500px;
  background-color: #222;
  display: flex;
  align-items: center;
  justify-content: center;
  color: #7f8c8d;
  font-weight: bold;
  border-radius: 20px;
}

.movie-main-title {
  font-size: 52px;
  font-weight: 900;
  letter-spacing: -1px;
  line-height: 1.2;
  margin: 0 0 20px 0;
}

.tagline {
  font-size: 20px;
  font-style: italic;
  color: #a4b0be;
  border-left: 3px solid #7f8c8d;
  padding-left: 15px;
  margin-bottom: 25px;
}

.meta-dashboard {
  display: flex;
  gap: 15px;
  flex-wrap: wrap;
  margin-bottom: 20px;
}

.badge {
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
  padding: 8px 16px;
  border-radius: 30px;
  font-weight: bold;
  font-size: 15px;
}

.rating {
  color: #ffa502;
  border-color: rgba(255, 165, 2, 0.3);
  background: rgba(255, 165, 2, 0.1);
}

.genres-wrapper {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-bottom: 30px;
}

.genre-tag {
  background-color: #1e272e;
  padding: 6px 14px;
  border-radius: 20px;
  font-size: 14px;
  border: 1px solid rgba(255, 255, 255, 0.1);
}

/* 재무 및 시놉시스 상자 */
.financial-box {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  background: rgba(0, 0, 0, 0.4);
  padding: 30px;
  border-radius: 16px;
  border: 1px solid rgba(255, 255, 255, 0.08);
}

.financial-item {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.f-label {
  font-size: 13px;
  font-weight: bold;
  color: #a4b0be;
}

.f-value {
  font-size: 28px;
  font-weight: 800;
  font-family: monospace;
}

.budget-color {
  color: #5352ed;
}

.revenue-color {
  color: #2ed573;
}

.synopsis-container {
  display: flex;
  flex-direction: column;
  margin-top: 30px;
}

.synopsis-title {
  font-size: 24px;
  font-weight: bold;
  border-left: 5px solid #ff4757;
  padding-left: 15px;
  margin: 0 0 15px 0;
}

.synopsis-text {
  font-size: 17px;
  line-height: 1.9;
  text-align: justify;
  color: #dcdde1;
  margin: 0;
}

.divider {
  border: 0;
  height: 1px;
  background: rgba(255, 255, 255, 0.2);
  margin: 30px 0;
}

.ai-section {
  margin: 30px 0;
  padding: 20px;
  border-radius: 12px;
  background: rgba(255, 255, 255, 0.1);
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.ai-btn {
  background: linear-gradient(45deg, #8a2be2, #4b0082);
  color: white;
  border: none;
  padding: 12px 20px;
  font-size: 16px;
  font-weight: bold;
  border-radius: 8px;
  cursor: pointer;
  width: 100%;
}

.ai-result-box {
  margin-top: 15px;
  padding: 15px;
  background: #222;
  border-left: 4px solid #8a2be2;
  color: #fff;
  line-height: 1.6;
}

/* 로딩 게이트, 에러 게이트 및 애니메이션 */
.full-screen-loading-gate,
.full-screen-error-gate {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  background-color: #0c1014;
  color: #ffffff;
  padding: 40px;
  text-align: center;
}

.loading-spinner {
  width: 60px;
  height: 60px;
  border: 5px solid rgba(255, 255, 255, 0.1);
  border-top-color: #ff4757;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin-bottom: 25px;
}

.loading-text,
.error-msg {
  font-weight: bold;
  font-size: 18px;
  color: #a4b0be;
}

.error-icon {
  font-size: 64px;
  margin-bottom: 20px;
}

.error-title {
  font-size: 32px;
  font-weight: 900;
  color: #ff4757;
  margin: 0 0 15px 0;
}

.error-return-btn {
  margin-top: 30px;
  padding: 14px 28px;
  background: #ff4757;
  color: white;
  border: none;
  border-radius: 30px;
  font-weight: bold;
  font-size: 16px;
  cursor: pointer;
  box-shadow: 0 4px 20px rgba(255, 71, 87, 0.4);
  transition: 0.3s;
}

.error-return-btn:hover {
  background: #ff6b81;
  transform: translateY(-2px);
}

@keyframes spin {
  from {
    transform: rotate(0deg);
  }
  to {
    transform: rotate(360deg);
  }
}

/* 반응형 디자인 */
@media (max-width: 900px) {
  .movie-hero-grid {
    grid-template-columns: 1fr;
    justify-items: center;
    gap: 40px;
  }

  .poster-zone {
    width: 280px;
  }

  .movie-main-title {
    font-size: 38px;
    text-align: center;
  }

  .tagline {
    text-align: center;
    border-left: none;
    padding-left: 0;
  }

  .meta-dashboard {
    justify-content: center;
  }

  .genres-wrapper {
    justify-content: center;
  }

  .financial-box {
    grid-template-columns: 1fr;
  }
}
</style>
