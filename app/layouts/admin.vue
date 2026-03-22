<script setup>
import { ref } from 'vue'

const isMobileMenuOpen = ref(false)
const isCollapsed = ref(false) // ตัวแปรควบคุมการพับ (Desktop)
</script>

<template>
  <div class="flex h-screen bg-gray-50 overflow-hidden">
      
      <AdminSidebar 
        :is-open="isMobileMenuOpen" 
        :is-collapsed="isCollapsed" 
        @close="isMobileMenuOpen = false" 
        @toggle-collapse="isCollapsed = !isCollapsed" 
      />
      
      <div class="flex-1 flex flex-col min-w-0 overflow-hidden">
        
        <AdminNavbar 
          @toggle-menu="isMobileMenuOpen = !isMobileMenuOpen" 
        />

      <!-- 3. Main Content Area -->
      <main class="flex-1 overflow-y-auto relative scrollbar-hide">
        <slot />
      </main>

      <!-- Overlay สำหรับมือถือ -->
      <div 
        v-if="isMobileMenuOpen" 
        @click="isMobileMenuOpen = false"
        class="fixed inset-0 bg-black bg-opacity-50 z-40 md:hidden transition-opacity"
      ></div>
    </div>
  </div>
</template>