<template>
  <q-page class="flex flex-center bg-judy-1">
    <div class="row justify-center text-center">
      <div class="col-12 col-sm-8">
        <q-img ref="logoRef" src="~assets/logo.png" class="logo-animation" />
      </div>
      <div class="col-12 col-sm-8 q-gutter-lg">
        <div ref="titleRef" class="text-h2 text-judy-2 intro_title title-animation">I'm Judy</div>
        <div ref="descRef" class="text-h6 about-description description-animation">
          <p>Judy Wang，畫畫和寫字的人。</p>
          <p>相信萬物皆有靈，喜歡用圖像和文字爲日常生活賦予生命力。</p>
        </div>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref, onMounted, onUnmounted } from 'vue'
import { gsap } from 'gsap'

// 元素引用
const logoRef = ref(null)
const titleRef = ref(null)
const descRef = ref(null)

let ctx = null
let shakeAnimation = null

onMounted(() => {
  // 等待 DOM 完全渲染
  setTimeout(() => {
    ctx = gsap.context(() => {
      // 設定初始狀態 - 所有元素都隱藏
      gsap.set('.logo-animation', {
        opacity: 0,
        scale: 0.5,
        y: -30,
      })

      gsap.set('.title-animation', {
        opacity: 0,
        y: 50,
        scale: 0.8,
      })

      gsap.set('.description-animation', {
        opacity: 0,
        y: 30,
      })

      // 創建時間軸動畫
      const tl = gsap.timeline()

      // Logo 動畫 - 從上方縮放淡入
      tl.to('.logo-animation', {
        opacity: 1,
        scale: 1,
        y: 0,
        duration: 1.2,
        ease: 'back.out(1.7)',
      })

        // 標題動畫 - 從下方淡入並縮放
        .to(
          '.title-animation',
          {
            opacity: 1,
            y: 0,
            scale: 1,
            duration: 1,
            ease: 'power2.out',
          },
          '-=0.6', // 與上一個動畫重疊 0.6 秒
        )

        // 描述文字動畫 - 從下方淡入
        .to(
          '.description-animation',
          {
            opacity: 1,
            y: 0,
            duration: 0.8,
            ease: 'power2.out',
          },
          '-=0.4', // 與上一個動畫重疊 0.4 秒
        )

        // 添加微妙的後續效果
        .to(
          '.title-animation',
          {
            textShadow: '0 2px 10px rgba(235, 140, 111, 0.3)',
            duration: 0.5,
            ease: 'power1.inOut',
          },
          '-=0.2',
        )

      // 🎯 Logo hover 效果 - 持續旋轉抖動
      const logoElement = document.querySelector('.logo-animation')

      logoElement.addEventListener('mouseenter', () => {
        // 停止之前的抖動動畫
        if (shakeAnimation) {
          shakeAnimation.kill()
        }

        // 創建持續的旋轉抖動效果
        shakeAnimation = gsap
          .timeline({ repeat: -1 })
          .to(logoElement, {
            rotation: 3,
            x: 2,
            duration: 0.1,
            ease: 'power2.inOut',
          })
          .to(logoElement, {
            rotation: -3,
            x: -2,
            duration: 0.1,
            ease: 'power2.inOut',
          })
          .to(logoElement, {
            rotation: 2,
            x: 1,
            duration: 0.1,
            ease: 'power2.inOut',
          })
          .to(logoElement, {
            rotation: -2,
            x: -1,
            duration: 0.1,
            ease: 'power2.inOut',
          })
          .to(logoElement, {
            rotation: 1,
            x: 0.5,
            duration: 0.1,
            ease: 'power2.inOut',
          })
          .to(logoElement, {
            rotation: -1,
            x: -0.5,
            duration: 0.1,
            ease: 'power2.inOut',
          })
      })

      logoElement.addEventListener('mouseleave', () => {
        // 停止抖動動畫
        if (shakeAnimation) {
          shakeAnimation.kill()
        }

        // 回到原始狀態
        gsap.to(logoElement, {
          rotation: 0,
          x: 0,
          scale: 1,
          duration: 0.3,
          ease: 'power2.out',
        })
      })
    })
  }, 100)
})

onUnmounted(() => {
  // 清理動畫
  if (shakeAnimation) {
    shakeAnimation.kill()
  }
  if (ctx) {
    ctx.revert()
  }
})
</script>

<style scoped>
.intro_title {
  font-family: 'Comic Sans MS', 'Comic Sans', cursive;
  font-weight: bold;
}

.about-description {
  font-family: 'Arial', sans-serif;
}

/* 動畫元素的初始狀態 */
.logo-animation,
.title-animation,
.description-animation {
  opacity: 0;
}

/* Logo 設定為可點擊狀態 */
.logo-animation {
  cursor: pointer;
}
</style>
