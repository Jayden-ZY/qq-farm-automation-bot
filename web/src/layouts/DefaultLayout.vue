<script setup lang="ts">
import { storeToRefs } from 'pinia'
import Sidebar from '@/components/Sidebar.vue'
import { useAppStore } from '@/stores/app'

const appStore = useAppStore()
const { sidebarOpen } = storeToRefs(appStore)
</script>

<template>
  <div class="w-screen flex overflow-hidden bg-gray-50 dark:bg-gray-900" style="height: 100dvh;">
    <!-- 全局水印 -->
    <div class="watermark-container">
      <div class="watermark" />
    </div>

    <!-- Mobile Sidebar Overlay -->
    <div
      v-if="sidebarOpen"
      class="fixed inset-0 z-40 bg-gray-900/50 backdrop-blur-sm transition-opacity lg:hidden"
      @click="appStore.closeSidebar"
    />

    <Sidebar />

    <main class="relative h-full min-w-0 flex flex-1 flex-col overflow-hidden">
      <!-- Top Bar (Mobile/Tablet only or for additional actions) -->
      <header class="h-16 flex shrink-0 items-center justify-between border-b border-gray-100 bg-white px-6 lg:hidden dark:border-gray-700/50 dark:bg-gray-800">
        <div class="text-lg font-bold">
          QQ农场智能助手
        </div>
        <button
          class="flex items-center justify-center rounded-lg p-2 text-gray-500 hover:bg-gray-100 dark:text-gray-400 dark:hover:bg-gray-700"
          @click="appStore.toggleSidebar"
        >
          <div class="i-carbon-menu text-xl" />
        </button>
      </header>

      <!-- Main Content Area -->
      <div class="flex flex-1 flex-col overflow-hidden">
        <div class="custom-scrollbar flex flex-1 flex-col overflow-y-auto p-2 md:p-6 sm:p-4">
          <RouterView v-slot="{ Component, route }">
            <Transition name="slide-fade" mode="out-in">
              <component :is="Component" :key="route.path" />
            </Transition>
          </RouterView>
        </div>
      </div>
    </main>
  </div>
</template>

<style scoped>
/* 水印样式 */
.watermark-container {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  pointer-events: none;
  z-index: 9999;
  overflow: hidden;
}

.watermark {
  position: absolute;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background-image: repeating-linear-gradient(
    0deg,
    transparent,
    transparent 80px,
    rgba(0, 0, 0, 0.02) 80px,
    rgba(0, 0, 0, 0.02) 81px
  ),
  repeating-linear-gradient(
    90deg,
    transparent,
    transparent 80px,
    rgba(0, 0, 0, 0.02) 80px,
    rgba(0, 0, 0, 0.02) 81px
  );
}

.watermark::before {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='240' height='120'%3E%3Ctext x='50%25' y='50%25' font-size='14' font-family='Arial, sans-serif' fill='rgba(128, 128, 128, 0.15)' text-anchor='middle' dominant-baseline='middle' transform='rotate(-20, 120, 60)'%3E开源项目，禁止倒卖！%3C/text%3E%3C/svg%3E");
  background-repeat: repeat;
  transform: rotate(-0deg);
}

.dark .watermark::before {
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='240' height='120'%3E%3Ctext x='50%25' y='50%25' font-size='14' font-family='Arial, sans-serif' fill='rgba(200, 200, 200, 0.1)' text-anchor='middle' dominant-baseline='middle' transform='rotate(-20, 120, 60)'%3E开源项目，禁止倒卖！%3C/text%3E%3C/svg%3E");
}

/* Slide Fade Transition */
.slide-fade-enter-active,
.slide-fade-leave-active {
  transition: all 0.2s ease-out;
}

.slide-fade-enter-from {
  opacity: 0;
  transform: translateY(10px);
}

.slide-fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}

.custom-scrollbar::-webkit-scrollbar {
  width: 6px;
  height: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: rgba(156, 163, 175, 0.3);
  border-radius: 3px;
}
.custom-scrollbar:hover::-webkit-scrollbar-thumb {
  background-color: rgba(156, 163, 175, 0.5);
}
</style>
