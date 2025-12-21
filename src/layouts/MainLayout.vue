<template>
  <q-layout view="hHr lpr ffr">
    <q-header bordered class="main-header bg-judy-1" height-hint="80">
      <q-toolbar class="q-pl-xs toolbar-content">
        <!-- Logo 區域 -->
        <q-toolbar-title class="logo-section">
          <q-btn flat dense to="/" class="logo-btn">
            <q-avatar class="logo-avatar">
              <img src="~assets/logo.png" alt="logo" />
            </q-avatar>
            <span class="logo-text text-judy-7">JUDY WANG ART</span>
          </q-btn>
        </q-toolbar-title>

        <!-- 導航選單區域 -->
        <div class="q-mr-xs">
          <!-- 小尺寸 -->
          <q-btn-dropdown class="lt-sm" flat dense color="judy-1" text-color="judy-7" label="選單">
            <q-list>
              <template v-for="item in navItems" :key="item.to">
                <q-item v-if="item.show !== false" :to="item.to" clickable v-close-popup>
                  <q-item-section>
                    <q-item-label>{{ item.label }}</q-item-label>
                  </q-item-section>
                </q-item>
              </template>
            </q-list>
          </q-btn-dropdown>

          <!-- 大尺寸 -->
          <q-tabs align="right" class="gt-xs" active-color="judy-7" indicator-color="judy-6">
            <template v-for="(item, index) in navItems" :key="item.to">
              <q-route-tab
                v-if="item.show !== false"
                :to="item.to"
                :label="item.label"
                exact
                class="nav-tab"
                :class="`nav-tab-${index + 1}`"
                no-caps
              />
            </template>
          </q-tabs>
        </div>

        <!-- 用戶操作區域 -->
        <div class="user-section">
          <q-btn flat no-caps class="user-btn bg-judy-3 text-white" @click="toggleRightDrawer">
            <q-icon
              :name="userStore.isLoggedIn ? 'person' : 'person_add'"
              size="20px"
              class="q-mr-xs"
            />
            <span class="user-text">
              {{ userStore.isLoggedIn ? userStore.account : '登入 / 註冊' }}
            </span>
          </q-btn>
        </div>
      </q-toolbar>
    </q-header>

    <!-- 右側抽屜 -->
    <q-drawer
      v-model="rightDrawerOpen"
      side="right"
      elevated
      overlay
      behavior="mobile"
      class="main-drawer bg-white"
      :width="360"
    >
      <div class="drawer-content">
        <UserProfile v-if="userStore.isLoggedIn" @closeDrawer="toggleRightDrawer" />
        <LoginForm
          v-if="!userStore.isLoggedIn && !showRegister"
          :toggleRegister="toggleRegister"
          @closeDrawer="toggleRightDrawer"
        />
        <RegisterForm v-if="showRegister" :toggleRegister="toggleRegister" />
      </div>
    </q-drawer>

    <!-- 主要內容區域 -->
    <q-page-container>
      <router-view :key="$route.fullPath" />
      <!-- 購物車浮動按鈕 -->
      <q-page-sticky
        v-if="$route.path === '/shopping' || $route.path.startsWith('/product')"
        position="top-right"
        :offset="fabPos"
        class="cart-fab"
      >
        <q-btn
          rounded
          no-caps
          icon="shopping_cart"
          class="cart-btn bg-judy-4 text-white"
          :disable="draggingFab"
          v-touch-pan.prevent.mouse="moveFab"
          label="結帳"
          :to="'/cart'"
        >
          <q-badge color="judy-5" text-color="white" rounded floating class="cart-badge">
            {{ userStore.cartTotal }}
          </q-badge>
        </q-btn>
      </q-page-sticky>
    </q-page-container>

    <!-- footer -->
    <q-footer class="bg-judy-2 text-black">
      <div>
        <!-- 社群連結 icon -->
        <div
          class="flex flex-row justify-center"
          :class="{
            'absolute-top-left': $q.screen.gt.sm,
          }"
        >
          <q-btn
            v-for="social in SocialLinks"
            :key="social.name"
            :href="social.url"
            target="_blank"
            flat
            round
            :icon="social.icon"
            size="md"
            color="white"
            :aria-label="`追蹤我們的 ${social.name}`"
          />
        </div>
        <!-- 版權 -->
        <div class="text-subtitle2 text-center">
          <div>© 2025 Judy Wang Art. All rights reserved.</div>
          <div>Website by BobKuo</div>
        </div>
      </div>
    </q-footer>
  </q-layout>
</template>

<script setup>
import { computed, ref } from 'vue'
import LoginForm from 'src/components/LoginForm.vue'
import RegisterForm from 'src/components/RegisterForm.vue'
import UserProfile from 'src/components/UserProfile.vue'
import { useUserStore } from 'src/stores/user'
// import MemberActions from '../components/MemberActions.vue'

const userStore = useUserStore()

const rightDrawerOpen = ref(false)
const showRegister = ref(false) // 是否顯示註冊表單

//
const navItems = computed(() => [
  // { to: '/daily', label: '365日常', show: true },
  { to: '/work', label: '作品集' },
  // { to: '/paper', label: '著色紙 下載', show: userStore.isLoggedIn },
  { to: '/shopping', label: '原畫及周邊' },
  { to: '/clips', label: '小短片' },
  { to: '/about', label: '關於我' },
  // { to: '/aboutsample', label: '關於我範例' },
  // { to: '/contact', label: '聯絡我' },
  // { to: '/admin/works', label: '管理後台', show: userStore.isLoggedIn && userStore.isAdmin },
])

// 切換右側側欄
const toggleRightDrawer = () => {
  rightDrawerOpen.value = !rightDrawerOpen.value
}

// 切換註冊表單顯示
const toggleRegister = () => {
  showRegister.value = !showRegister.value
}

// 購物車
const fabPos = ref([50, 18])
const draggingFab = ref(false)
const moveFab = (event) => {
  draggingFab.value = event.isFirst !== true && event.isFinal !== true

  fabPos.value = [fabPos.value[0] - event.delta.x, fabPos.value[1] + event.delta.y]
  // fabPos.value = [fabPos.value[0] - event.delta.x, fabPos.value[1] - event.delta.y]
}

// Footer 社群媒體連結
const SocialLinks = ref([
  { name: 'Instagram', icon: 'fab fa-instagram', url: 'https://www.instagram.com/judywang.art/' },
  { name: 'Threads', icon: 'fab fa-threads', url: 'https://www.threads.com/@judywang.art/' },
  { name: 'TikTok', icon: 'fab fa-tiktok', url: 'https://www.tiktok.com/@judywang_art/' },
  { name: 'Facebook', icon: 'fab fa-facebook-f', url: 'https://www.facebook.com/judywangswork/' },
])
</script>
<style scoped>
/* 主要 Header 樣式 */
/* .main-header {
  border-bottom: 1px solid rgba(235, 140, 111, 0.3);
  box-shadow: 0 2px 12px rgba(0, 0, 0, 0.1);
  backdrop-filter: blur(8px);
} */

/* .toolbar-content {
  min-height: 80px;
  display: flex;
  align-items: center;
  gap: 24px;
} */

/* Logo 區域 */
/* .logo-section {
  flex-shrink: 0;
} */

.logo-btn {
  /* padding: 8px 16px; */
  border-radius: 12px;
  transition: all 0.3s ease;
}

.logo-btn:hover {
  background-color: rgba(235, 140, 111, 0.1);
  transform: translateY(-1px);
}

/* .logo-avatar {
  border: 2px solid rgba(235, 140, 111, 0.3);
} */

.logo-text {
  font-weight: 600;
  font-size: 1.1rem;
  margin-left: 12px;
  letter-spacing: 0.5px;
}

/* 導航區域 */

.nav-tab {
  /* border-radius: 12px; */
  /* margin: 0 4px; */
  min-height: 44px;
  font-weight: 500;
  /* font-size: 0.95rem; */
  transition: all 0.3s ease;
  color: rgba(58, 78, 61, 0.8);
}

.nav-tab:hover {
  background-color: rgba(235, 140, 111, 0.2);
  color: rgba(58, 78, 61, 1);
  transform: translateY(-1px);
}

.nav-tab.q-tab--active {
  background-color: rgba(235, 140, 111, 0.3);
  color: rgba(58, 78, 61, 1);
  font-weight: 600;
}

/* 不同導航項目的漸層色彩 */
.nav-tab-1:hover {
  background-color: rgba(235, 140, 111, 0.2);
} /* judy-2 */
.nav-tab-2:hover {
  background-color: rgba(154, 198, 219, 0.2);
} /* judy-3 */
.nav-tab-3:hover {
  background-color: rgba(122, 184, 163, 0.2);
} /* judy-4 */
.nav-tab-4:hover {
  background-color: rgba(217, 164, 65, 0.2);
} /* judy-5 */
.nav-tab-5:hover {
  background-color: rgba(170, 170, 170, 0.2);
} /* judy-6 */

/* 用戶操作區域 */
.user-section {
  flex-shrink: 0;
}

/* .user-btn {
  padding: 10px 20px;
  border-radius: 24px;
  min-height: 44px;
  transition: all 0.3s ease;
  border: 1px solid rgba(154, 198, 219, 0.3);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
} */

.user-btn:hover {
  background-color: rgba(154, 198, 219, 0.9) !important;
  transform: translateY(-2px);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
}

.user-text {
  font-weight: 500;
  font-size: 0.9rem;
  max-width: 120px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

/* 右側抽屜 */
.main-drawer {
  border-left: 1px solid rgba(235, 140, 111, 0.2);
}

.drawer-content {
  height: 100%;
  overflow-y: auto;
}

.toggle-btn {
  margin: 8px;
  background-color: rgba(170, 170, 170, 0.1);
  border-radius: 50%;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.toggle-btn:hover {
  background-color: rgba(255, 255, 255, 0.9);
  border: 1px solid rgba(170, 170, 170, 0.3);
  transform: scale(1.05);
}

/* 購物車浮動按鈕 */
.cart-fab {
  z-index: 1000;
}

.cart-btn {
  padding: 12px 20px;
  border-radius: 28px;
  min-height: 56px;
  font-weight: 600;
  font-size: 0.95rem;
  border: 2px solid rgba(122, 184, 163, 0.3);
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.15);
  transition: all 0.3s ease;
}

.cart-btn:hover {
  background-color: rgba(122, 184, 163, 0.9) !important;
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.2);
}

.cart-badge {
  font-weight: 600;
  font-size: 0.75rem;
}

/* 響應式設計 */
@media (max-width: 1024px) {
  .logo-text {
    font-size: 1rem;
  }

  .user-text {
    display: none;
  }
}

@media (max-width: 600px) {
  .toolbar-content {
    min-height: 64px;
    /* gap: 12px;
    padding: 0 16px; */
  }

  .user-btn {
    /* padding: 8px 16px; */
    min-height: 40px;
  }

  .cart-btn {
    /* padding: 10px 16px; */
    min-height: 48px;
    font-size: 0.85rem;
  }
}

/* 動畫效果 */
@keyframes fadeInDown {
  from {
    opacity: 0;
    transform: translateY(-20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.main-header {
  animation: fadeInDown 0.6s ease-out;
}

/* 自定義變數覆蓋 */
.main-header {
  height: var(--app-header-height, 80px);
}

.main-header .q-toolbar {
  min-height: var(--app-header-height, 80px);
}

/* Judy 顏色系統懸停效果 */
.nav-tab-1.q-tab--active {
  background-color: rgba(235, 140, 111, 0.3);
}
.nav-tab-2.q-tab--active {
  background-color: rgba(154, 198, 219, 0.3);
}
.nav-tab-3.q-tab--active {
  background-color: rgba(122, 184, 163, 0.3);
}
.nav-tab-4.q-tab--active {
  background-color: rgba(217, 164, 65, 0.3);
}
.nav-tab-5.q-tab--active {
  background-color: rgba(170, 170, 170, 0.3);
}
</style>
