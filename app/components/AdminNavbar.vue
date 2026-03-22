<script setup>
defineProps(['isMobileMenuOpen'])
// คงไว้แค่ toggleMenu สำหรับเปิดเมนูในมือถือ
defineEmits(['toggleMenu']) 
</script>

<template>
  <header class="bg-white border-b border-gray-200 sticky top-0 z-40 h-16 flex items-center px-4 md:px-8 justify-between">
    <div class="flex items-center gap-4">
      <!-- ปุ่ม 3 ขีด สำหรับ มือถือ เท่านั้น -->
      <button 
        @click="$emit('toggleMenu')"
        class="md:hidden p-2 text-gray-500 hover:bg-gray-100 rounded-lg transition"
      >
        <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
          <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" />
        </svg>
      </button>
      
      <!-- ซ่อนชื่อในจอ Desktop เพราะแสดงใน Sidebar แล้ว -->
      <h1 class="font-bold text-lg text-gray-800 md:hidden">พอพิน</h1>
    </div>

    <!-- ส่วนแสดงโปรไฟล์มุมขวาบน -->
    <div class="flex items-center gap-3">
      <div class="text-right hidden sm:block">
        <p class="text-xs font-bold text-gray-800">Admin Somying</p>
        <p class="text-[10px] text-gray-400">Super Admin</p>
      </div>
      <div class="w-10 h-10 rounded-full bg-[#ff8fa3] flex items-center justify-center text-white font-bold shadow-sm">
        AS
      </div>
    </div>
  </header>
</template>