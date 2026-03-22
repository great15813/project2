<script setup>
// นำเข้า useRoute เพื่อดึง URL ปัจจุบันมาตรวจสอบว่าเราอยู่หน้าไหน (ใช้สำหรับทำแถบเมนูสว่าง/Active)
import { useRoute } from 'vue-router'

// รับค่า Props จาก Layout (admin.vue) เพื่อเช็คว่า Sidebar กำลังถูกเปิดหรือพับอยู่
defineProps(['isOpen', 'isCollapsed'])

// ประกาศ Event ที่จะส่งกลับไปให้ Layout จัดการ (เช่น การปิดเมนูในมือถือ หรือ การพับเมนูในจอคอม)
defineEmits(['close', 'toggleCollapse'])

// ประกาศตัวแปร route เพื่อเรียกใช้งาน URL ปัจจุบัน
const route = useRoute()
</script>

<template>
  <aside 
    class="fixed inset-y-0 left-0 z-50 bg-white border-r border-gray-200 transition-all duration-300 ease-in-out md:static md:inset-0 h-screen overflow-y-auto overflow-x-hidden flex flex-col"
    :class="[
      isOpen ? 'translate-x-0' : '-translate-x-full md:translate-x-0',
      isCollapsed ? 'md:w-20' : 'md:w-64'
    ]"
  >
    <div class="h-16 border-b border-gray-200 px-4 flex items-center sticky top-0 bg-white z-10" :class="isCollapsed ? 'justify-center' : 'justify-between'">
      
      <div v-if="!isCollapsed" class="flex items-center gap-3 overflow-hidden">
        <div class="w-8 h-8 bg-[#ff8fa3] rounded-lg flex-shrink-0 flex items-center justify-center text-white font-bold">พ</div>
        <h1 class="font-bold text-lg text-gray-800 whitespace-nowrap">พอพิน คลังสินค้า</h1>
      </div>
      
      <button @click="$emit('toggleCollapse')" class="hidden md:flex p-2 text-gray-400 hover:bg-gray-100 rounded-lg transition items-center justify-center flex-shrink-0">
        <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4 6h16M4 12h16M4 18h16" /></svg>
      </button>
      
      <button @click="$emit('close')" class="md:hidden p-2 text-gray-400 hover:bg-gray-100 rounded-lg">
        <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
      </button>

    </div>

    <nav class="px-3 pb-6 flex-1 space-y-6 flex flex-col mt-6">
      
      <div>
        <p v-if="!isCollapsed" class="px-3 text-[10px] font-bold text-gray-400 uppercase tracking-widest mb-2 transition-opacity">Main Menu</p>
        
        <ul class="space-y-1">
        <!---- เมนูหลัก -->
          <li>
            <NuxtLink to="/admin" 
              class="group flex items-center px-3 py-3 rounded-xl transition relative"
              :class="[
                isCollapsed ? 'justify-center' : 'gap-3', 
                route.path === '/admin' ? 'bg-pink-50 text-[#ff8fa3] font-bold' : 'text-gray-600 hover:bg-gray-50'
              ]"
            >
              <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6" /></svg>
              <span v-if="!isCollapsed" class="whitespace-nowrap">Dashboard</span>
              <span v-if="isCollapsed" class="absolute left-14 bg-gray-800 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">Dashboard</span>
            </NuxtLink>
          </li>
          <!---- เมนูจัดการคำสั่งซื้อ -->
          <li>
            <NuxtLink to="/admin/orders" 
              class="group flex items-center px-3 py-3 rounded-xl transition relative"
              :class="[
                isCollapsed ? 'justify-center' : 'gap-3', 
                route.path.startsWith('/admin/order') ? 'bg-pink-50 text-[#ff8fa3] font-bold' : 'text-gray-600 hover:bg-gray-50'
              ]"
            >
              <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" /></svg>
              <span v-if="!isCollapsed" class="whitespace-nowrap">จัดการคำสั่งซื้อ</span>
              <span v-if="isCollapsed" class="absolute left-14 bg-gray-800 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">จัดการคำสั่งซื้อ</span>
            </NuxtLink>
          </li>

          <!---- เมนูจัดการสินค้า -->
          <li>
            <NuxtLink to="/admin/products" 
              class="group flex items-center px-3 py-3 rounded-xl transition relative"
              :class="[
                isCollapsed ? 'justify-center' : 'gap-3', 
                route.path.startsWith('/admin/products') ? 'bg-pink-50 text-[#ff8fa3] font-bold' : 'text-gray-600 hover:bg-gray-50'
              ]"
            >
              <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4" /></svg>
              <span v-if="!isCollapsed" class="whitespace-nowrap">สินค้า & สต๊อก</span>
              <span v-if="isCollapsed" class="absolute left-14 bg-gray-800 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">สินค้า & สต๊อก</span>
            </NuxtLink>
          </li>

          <!---- เมนูจัดการวัตถุดิบ -->
          <li>
            <NuxtLink to="/admin/materials" 
              class="group flex items-center px-3 py-3 rounded-xl transition relative"
              :class="[
                isCollapsed ? 'justify-center' : 'gap-3', 
                route.path.startsWith('/admin/materials') ? 'bg-pink-50 text-[#ff8fa3] font-bold' : 'text-gray-600 hover:bg-gray-50'
              ]"
            >
              <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 11H5m14 0a2 2 0 012 2v6a2 2 0 01-2 2H5a2 2 0 01-2-2v-6a2 2 0 012-2m14 0V9a2 2 0 00-2-2M5 11V9a2 2 0 012-2m0 0V5a2 2 0 012-2h6a2 2 0 012 2v2M7 7h10" /></svg>
              <span v-if="!isCollapsed" class="whitespace-nowrap">จัดการวัตถุดิบ</span>
              <span v-if="isCollapsed" class="absolute left-14 bg-gray-800 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">จัดการวัตถุดิบ</span>
            </NuxtLink>
          </li>

          <!---- เมนูประวัติการเคลื่อนไหว -->
          <li>
            <NuxtLink to="/admin/movements" 
              class="group flex items-center px-3 py-3 rounded-xl transition relative"
              :class="[
                isCollapsed ? 'justify-center' : 'gap-3', 
                route.path.startsWith('/admin/movements') ? 'bg-pink-50 text-[#ff8fa3] font-bold' : 'text-gray-600 hover:bg-gray-50'
              ]"
            >
              <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 5H7a2 2 0 00-2 2v12a2 2 0 002 2h10a2 2 0 002-2V7a2 2 0 00-2-2h-2M9 5a2 2 0 002 2h2a2 2 0 002-2M9 5a2 2 0 012-2h2a2 2 0 012 2m-6 9l2 2 4-4" /></svg>
              <span v-if="!isCollapsed" class="whitespace-nowrap">ประวัติการเคลื่อนไหว</span>
              <span v-if="isCollapsed" class="absolute left-14 bg-gray-800 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">ประวัติการเคลื่อนไหว</span>
            </NuxtLink>
          </li>

          
          <!---- เมนูผู้ใช้งาน & สิทธิ์ -->
          <li>
            <NuxtLink to="/admin/members" 
              class="group flex items-center px-3 py-3 rounded-xl transition relative"
              :class="[
                isCollapsed ? 'justify-center' : 'gap-3', 
                route.path.startsWith('/admin/members') ? 'bg-pink-50 text-[#ff8fa3] font-bold' : 'text-gray-600 hover:bg-gray-50'
              ]"
            >
              <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4.354a4 4 0 110 5.292M15 21H3v-1a6 6 0 0112 0v1zm0 0h6v-1a6 6 0 00-9-5.197M13 7a4 4 0 11-8 0 4 4 0 018 0z" /></svg>
              <span v-if="!isCollapsed" class="whitespace-nowrap">ผู้ใช้งาน & สิทธิ์</span>
              <span v-if="isCollapsed" class="absolute left-14 bg-gray-800 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">ผู้ใช้งาน & สิทธิ์</span>
            </NuxtLink>
          </li>
          
        </ul>
      </div>

      <div class="flex-1"></div>

      <div class="pt-4 border-t border-gray-100">
        <NuxtLink to="/" class="group flex items-center px-3 py-3 rounded-xl text-red-500 hover:bg-red-50 transition relative" :class="isCollapsed ? 'justify-center' : 'gap-3'">
          <svg class="w-6 h-6 flex-shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17 16l4-4m0 0l-4-4m4 4H7m6 4v1a3 3 0 01-3 3H6a3 3 0 01-3-3V7a3 3 0 013-3h4a3 3 0 013 3v1" /></svg>
          <span v-if="!isCollapsed" class="whitespace-nowrap">ออกจากระบบ</span>
          <span v-if="isCollapsed" class="absolute left-14 bg-red-600 text-white text-xs px-2 py-1 rounded opacity-0 group-hover:opacity-100 pointer-events-none transition z-50 whitespace-nowrap">ออกจากระบบ</span>
        </NuxtLink>
      </div>
      
    </nav>
  </aside>
</template>