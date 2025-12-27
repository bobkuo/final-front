<template>
  <div class="projects-section">
    <div class="container">
      <div class="q-pa-md relative-position" style="min-height: 50vh">
        <!-- 載入狀態 -->
        <q-inner-loading
          :showing="loading"
          label="載入作品中，請稍候..."
          label-class="text-judy-7"
          color="judy-2"
          size="80px"
        />

        <!-- Carousel -->
        <template v-if="!loading">
          <q-carousel
            v-model="slide"
            transition-prev="slide-right"
            transition-next="slide-left"
            animated
            control-color="primary"
            class="rounded-borders"
          >
            <q-carousel-slide
              v-for="series in allSeries"
              :key="series._id"
              :name="series._id"
              :img-src="series.cover"
            >
              <div class="absolute-bottom custom-caption">
                <div class="text-h2">{{ series.name }}</div>
                <div class="text-subtitle1">{{ series.description }}</div>
              </div>
            </q-carousel-slide>
          </q-carousel>

          <!-- Carousel 控制按鈕 -->
          <div v-if="!loading && allSeries.length > 0" class="row justify-center q-mt-md">
            <q-btn-toggle
              v-model="slide"
              :options="carouselOptions"
              toggle-color="judy-2"
              color="white"
              text-color="judy-2"
              unelevated
            />
          </div>
        </template>
      </div>

      <!-- 顯示當前選中系列的作品 -->
      <div v-if="!loading && currentSeries" class="row justify-center">
        <div class="col-12">
          <h2 class="section-title">{{ currentSeries.name }} - 作品集</h2>
        </div>

        <!-- 如果當前系列有作品 -->
        <div v-if="currentWorks.length > 0" class="projects-grid col-12 col-md-10">
          <div
            v-for="(work, index) in currentWorks"
            :key="work._id"
            :id="`project-${work._id}`"
            class="project-section"
          >
            <WorkCard :project="transformWorkToProject(work)" :is-reverse="index % 2 === 1" />
          </div>
        </div>

        <!-- 如果當前系列沒有作品 -->
        <div v-else class="no-works-container">
          <div class="no-works-message">
            <q-icon name="image" size="4rem" color="grey-5" />
            <h3>此系列暫無作品</h3>
            <p>請選擇其他系列查看作品</p>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted, computed, watch, nextTick } from 'vue'
import { useRoute } from 'vue-router'
import { useQuasar } from 'quasar'
import { gsap } from 'gsap'
import { ScrollTrigger } from 'gsap/ScrollTrigger'
import WorkCard from 'src/components/WorkCard.vue'
import seriesService from 'src/services/series'
import workService from 'src/services/work'

gsap.registerPlugin(ScrollTrigger)

const route = useRoute()
const $q = useQuasar()

// 響應式資料
const allSeries = ref([])
const allWorks = ref([])
const loading = ref(true)
const slide = ref('')

// 計算屬性：生成 carousel 控制按鈕選項
const carouselOptions = computed(() => {
  return allSeries.value.map((series) => ({
    label: series.name,
    value: series._id,
  }))
})

// 計算屬性：當前選中的系列
const currentSeries = computed(() => {
  return allSeries.value.find((series) => series._id === slide.value) // slide 為選到的系列 ID
})

// 計算屬性：當前系列的作品
const currentWorks = computed(() => {
  return allWorks.value.filter((work) => work.category === slide.value) // slide 為選到的系列 ID
})

// 將 work 物件轉換為 WorkCard 元件需要的 project 格式
const transformWorkToProject = (work) => {
  return {
    id: work._id,
    title: work.name,
    category: currentSeries.value?.name || '未分類', // 使用系列名稱作為分類
    description: work.content || '暫無描述',
    image: work.images?.[0] || 'https://via.placeholder.com/600x400?text=No+Image', // 使用第一張圖片
    tags: work.tags || [],
    statistics: work.statistics || {},
  }
}

// 🔗 處理錨點滾動到指定作品
const handleAnchorScroll = async () => {
  const hash = route.hash
  if (hash && hash.startsWith('#project-')) {
    // 提取作品 ID
    const projectId = hash.replace('#project-', '')

    // 先找到該作品屬於哪個系列
    const targetWork = allWorks.value.find((work) => work._id === projectId)

    if (targetWork) {
      // 直接切換到目標系列（在渲染前完成）
      slide.value = targetWork.category

      // 等待 DOM 更新完成後再滾動
      await nextTick()
      setTimeout(() => {
        scrollToProject(projectId, targetWork)
      }, 300)
    } else {
      // 找不到作品
      $q.notify({
        type: 'warning',
        message: '找不到指定的作品',
        position: 'top',
        timeout: 2000,
        icon: 'warning',
      })
    }
  }
}

// 🎯 滾動到指定作品並高亮
const scrollToProject = (projectId, work) => {
  const targetElement = document.getElementById(`project-${projectId}`)

  if (targetElement) {
    // 計算元素的中心點位置
    const elementRect = targetElement.getBoundingClientRect()
    const elementHeight = elementRect.height
    const viewportHeight = window.innerHeight

    // 計算滾動位置（考慮 header）
    const headerHeight = 50 // 根據您的 header 高度調整, q-header預設為50px

    // 計算元素在頁面中的絕對位置
    const elementPosition = elementRect.top + window.pageYOffset

    // 計算讓元素中心在畫面中心的滾動位置
    const offsetPosition = elementPosition - headerHeight - viewportHeight / 2 + elementHeight / 2

    // console.log('滾動到作品:', projectId)
    // console.log('元素高度:', elementHeight)
    // console.log('視窗高度:', viewportHeight)
    // console.log('目標滾動位置:', offsetPosition)

    // 平滑滾動到目標位置
    window.scrollTo({
      top: Math.max(0, offsetPosition),
      behavior: 'smooth',
    })

    // 添加高亮效果
    // highlightProject(targetElement)

    // 顯示通知
    $q.notify({
      type: 'info',
      message: `正在查看：${work.name}`,
      position: 'top',
      timeout: 3000,
      icon: 'visibility',
      actions: [
        {
          label: '關閉',
          color: 'white',
          handler: () => {},
        },
      ],
    })
  }
}

// ✨ 高亮作品效果
// const highlightProject = (element) => {
//   // 添加高亮 class
//   element.classList.add('highlighted')

//   // 3 秒後移除高亮效果
//   setTimeout(() => {
//     element.classList.remove('highlighted')
//   }, 3000)
// }

// 🔍 根據作品 ID 查找所屬系列
// const findSeriesByWorkId = (workId) => {
//   const work = allWorks.value.find(w => w._id === workId)
//   if (work) {
//     return allSeries.value.find(s => s._id === work.category)
//   }
//   return null
// }

// 監聽路由變化
watch(
  () => route.hash,
  (newHash, oldHash) => {
    if (newHash && newHash !== oldHash && newHash.startsWith('#project-')) {
      handleAnchorScroll()
    }
  },
)

// 監聽 slide 變化，可以添加切換動畫
watch(slide, (newSlide, oldSlide) => {
  if (newSlide !== oldSlide && newSlide) {
    console.log('切換到系列:', currentSeries.value?.name)
    console.log('該系列作品數量:', currentWorks.value.length)

    // 可以在這裡添加切換動畫
    // animateWorksTransition()
  }
})

// 作品切換動畫
// const animateWorksTransition = () => {
//   // 等待 DOM 更新後執行動畫
//   setTimeout(() => {
//     gsap.fromTo(
//       '.projects-grid .work-card',
//       {
//         y: 50,
//         opacity: 0,
//       },
//       {
//         y: 0,
//         opacity: 1,
//         duration: 0.6,
//         stagger: 0.1,
//         ease: 'power2.out',
//       },
//     )
//   }, 50)
// }

// const projects = ref([
//   {
//     id: 1,
//     title: 'Mountain Resort Branding',
//     category: 'Branding & Identity',
//     description:
//       'Complete brand identity design for a luxury mountain resort, including logo design, color palette, and brand guidelines.',
//     image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=600&h=400&fit=crop',
//   },
//   {
//     id: 2,
//     title: 'Forest Conservation Website',
//     category: 'Web Design',
//     description:
//       'Responsive website design for an environmental organization focused on forest conservation and sustainability initiatives.',
//     image: 'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?w=600&h=400&fit=crop',
//   },
//   {
//     id: 3,
//     title: 'Nature Photography Portfolio',
//     category: 'Portfolio Design',
//     description:
//       'Clean and minimalist portfolio website showcasing stunning nature photography with smooth transitions and galleries.',
//     image: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=600&h=400&fit=crop',
//   },
//   {
//     id: 4,
//     title: 'Outdoor Adventure App',
//     category: 'UI/UX Design',
//     description:
//       'Mobile app design for outdoor enthusiasts, featuring trail maps, weather updates, and community features.',
//     image: 'https://images.unsplash.com/photo-1441974231531-c6227db76b6e?w=600&h=400&fit=crop',
//   },
// ])

const getSeries = async () => {
  try {
    loading.value = true
    const response = await seriesService.get()
    allSeries.value = response.data.series

    // 設定預設的 slide（第一個系列）
    if (allSeries.value.length > 0) {
      slide.value = allSeries.value[0]._id
    }
  } catch (error) {
    console.error('Error fetching series:', error)
  } finally {
    loading.value = false
  }
}

const getWorks = async () => {
  try {
    loading.value = true
    const response = await workService.get()
    allWorks.value = response.data.works
  } catch (error) {
    console.error('Error fetching works:', error)
  } finally {
    loading.value = false
  }
}

onMounted(async () => {
  // 頁面載入時呼叫 API
  await getSeries()
  await getWorks()

  // 在渲染前處理錨點切換
  if (route.hash && route.hash.startsWith('#project-')) {
    handleAnchorScroll()
  } else {
    // 如果沒有錨點，設定預設的第一個系列
    if (allSeries.value.length > 0) {
      slide.value = allSeries.value[0]._id
    }
  }

  // 滾動觸發動畫
  // gsap.fromTo(
  //   '.project-item',
  //   {
  //     y: 100,
  //     opacity: 0,
  //   },
  //   {
  //     y: 0,
  //     opacity: 1,
  //     duration: 1,
  //     stagger: 0.2,
  //     ease: 'power2.out',
  //     scrollTrigger: {
  //       trigger: '.projects-grid',
  //       start: 'top 80%',
  //       end: 'bottom 20%',
  //       toggleActions: 'play none none reverse',
  //     },
  //   },
  // )
  // 圖片視差效果
  // gsap.utils.toArray('.project-image img').forEach((img) => {
  //   gsap.fromTo(
  //     img,
  //     { scale: 1.2 },
  //     {
  //       scale: 1,
  //       duration: 1.5,
  //       ease: 'power2.out',
  //       scrollTrigger: {
  //         trigger: img.closest('.project-item'),
  //         start: 'top 90%',
  //         end: 'bottom 10%',
  //         scrub: 1,
  //       },
  //     },
  //   )
  // })
})

// const handleViewProject = (project) => {
//   console.log('View project:', project)
//   // 這裡可以添加導航到項目詳情頁的邏輯
//   // 例如: router.push(`/projects/${project.id}`)
// }
</script>

<style scoped lang="scss">
.projects-section {
  // padding: 120px 0;
  // background: linear-gradient(135deg, #f8f9fa 0%, #e9ecef 100%);
  min-height: 100vh;
}

.container {
  max-width: 1920px;
  margin: 0 auto;
  padding: 0 20px;
}

.section-title {
  font-size: clamp(2.5rem, 5vw, 4rem);
  font-weight: 700;
  text-align: center;
  margin-bottom: 80px;
  color: #2d3436;
  position: relative;

  &::after {
    content: '';
    position: absolute;
    bottom: -20px;
    left: 50%;
    transform: translateX(-50%);
    width: 80px;
    height: 4px;
    background: linear-gradient(90deg, #6c5ce7, #a29bfe);
    border-radius: 2px;
  }
}

.projects-grid {
  // display: flex;
  // flex-direction: column;
  gap: 60px;
}

// 無作品時的樣式
.no-works-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 400px;

  .no-works-message {
    text-align: center;
    color: #636e72;

    h3 {
      margin: 1rem 0 0.5rem 0;
      font-size: 1.5rem;
      font-weight: 600;
    }

    p {
      margin: 0;
      font-size: 1rem;
      opacity: 0.8;
    }
  }
}

.custom-caption {
  text-align: center;
  padding: 12px;
  color: white;
  background-color: rgba(0, 0, 0, 0.3);
}

// 🎯 作品區塊樣式
.project-section {
  margin-bottom: 80px;
  scroll-margin-top: 50px; /* 為 header 和 carousel 預留空間 */
  transition: all 0.3s ease;
}

/* ✨ 高亮效果動畫 */
// .project-section.highlighted {
//   animation: highlight 3s ease-in-out;
//   border-radius: 20px;
//   padding: 20px;
//   margin: 20px 0;
// }

// @keyframes highlight {
//   0% {
//     background-color: transparent;
//     transform: scale(1);
//   }
//   25% {
//     background-color: rgba(235, 140, 111, 0.15);
//     transform: scale(1.02);
//     box-shadow: 0 10px 30px rgba(235, 140, 111, 0.2);
//   }
//   50% {
//     background-color: rgba(235, 140, 111, 0.1);
//     transform: scale(1.01);
//     box-shadow: 0 8px 25px rgba(235, 140, 111, 0.15);
//   }
//   75% {
//     background-color: rgba(235, 140, 111, 0.05);
//     transform: scale(1.005);
//     box-shadow: 0 5px 15px rgba(235, 140, 111, 0.1);
//   }
//   100% {
//     background-color: transparent;
//     transform: scale(1);
//     box-shadow: none;
//   }
// }

// /* 🎯 為分享連結訪問者添加特殊樣式 */
// .project-section:target {
//   animation: highlight 3s ease-in-out;
// }

// /* 📱 載入動畫 */
// .project-section {
//   opacity: 0;
//   animation: fadeInUp 0.6s ease-out forwards;
// }

// .project-section:nth-child(1) {
//   animation-delay: 0.1s;
// }
// .project-section:nth-child(2) {
//   animation-delay: 0.2s;
// }
// .project-section:nth-child(3) {
//   animation-delay: 0.3s;
// }
// .project-section:nth-child(4) {
//   animation-delay: 0.4s;
// }
// .project-section:nth-child(5) {
//   animation-delay: 0.5s;
// }

// @keyframes fadeInUp {
//   from {
//     opacity: 0;
//     transform: translateY(30px);
//   }
//   to {
//     opacity: 1;
//     transform: translateY(0);
//   }
// }

// 響應式設計
@media (max-width: 1024px) {
  .projects-section {
    padding: 80px 0;
  }

  .projects-grid {
    gap: 20px;
  }
}

@media (max-width: 768px) {
  .projects-section {
    padding: 60px 0;
  }

  .section-title {
    margin-bottom: 60px;
  }

  .projects-grid {
    gap: 0px;
  }

  .project-section {
    margin-bottom: 60px;
    scroll-margin-top: 300px;
  }

  .project-section.highlighted {
    padding: 10px;
    margin: 10px 0;
  }
}
</style>
