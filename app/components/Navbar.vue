<script setup>
import { ref } from 'vue'

const isMobileMenuOpen = ref(false)

// [BACKEND: ดึงสถานะการล็อกอินจริงจาก Supabase]
// เช่น const user = useSupabaseUser()
// const isLoggedIn = computed(() => !!user.value)

// ตัวแปรจำลอง: ตอนนี้สมมติว่า "ยังไม่ได้ล็อกอิน (false)"
const isLoggedIn = ref(false) 
</script>

<template>
  <div>
    <!-- แถบ Navbar หลัก -->
    <header class="bg-[#ff8fa3] h-16 md:h-20 relative flex items-center px-4 md:px-8 z-50 shadow-sm">
      
      <!-- ซ้าย: โลโก้ (Overlap ออกมานอกกรอบ) -->
      <NuxtLink to="/" class="absolute left-4 md:left-8 top-1 md:top-2 w-16 h-16 md:w-28 md:h-28 z-50">
        <!-- เปลี่ยน /path-to-your-logo.png เป็นที่อยู่ไฟล์รูปโลโก้ของคุณ -->
        <img 
          src="../assets/logo/Logo2.png" 
          alt="Shop Logo" 
          class="w-full h-full rounded-full border-[3px] border-white shadow-md bg-pink-50 object-cover" 
        />
      </NuxtLink>
      
      <!-- จอ PC -->
      <!-- กลาง: เมนู (จอ PC) -->
      <!-- ใช้ md:pl-28 เพื่อเว้นระยะไม่ให้เมนูทับกับโลโก้ที่ใหญ่ขึ้น -->
      <nav class="hidden md:flex flex-1 justify-center space-x-6 lg:space-x-10 text-white font-bold text-lg md:pl-28">
        <NuxtLink to="/" class="hover:text-pink-200 transition"active-class="active">Home</NuxtLink>
        <!-- ตัวอย่างคลาส Active (มีเส้นใต้) -->
        <NuxtLink to="/about" class="hover:text-pink-200 transition" active-class="active">About us</NuxtLink>
        <NuxtLink to="/productscategories" class="hover:text-pink-200 transition" active-class="active">Product</NuxtLink>
        <NuxtLink to="/faq" class="hover:text-pink-200 transition" active-class="active">FAQ</NuxtLink>
        <NuxtLink to="/contact" class="hover:text-pink-200 transition" active-class="active">Contact us</NuxtLink>
      </nav>

      <!-- ขวา: ไอคอนโปรไฟล์, ตะกร้า และปุ่มเมนูมือถือ -->
      <div class="flex items-center space-x-4 md:space-x-6 text-white ml-auto z-50">
        
        <!-- ไอคอนโปรไฟล์ (ระบบจะเลือกพาไปหน้า login หรือ profile อัตโนมัติ) -->
        <NuxtLink 
          :to="isLoggedIn ? '/profile' : '/login'" 
          class="hover:scale-110 transition duration-200"
          title="บัญชีผู้ใช้"
        >
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 md:w-8 md:h-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M5.121 17.804A13.937 13.937 0 0112 16c2.5 0 4.847.655 6.879 1.804M15 10a3 3 0 11-6 0 3 3 0 016 0zm6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
        </NuxtLink>

        <!-- ไอคอนตะกร้าสินค้า -->
        <NuxtLink to="/cart" class="hover:scale-110 transition duration-200 relative">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-6 h-6 md:w-8 md:h-8" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path stroke-linecap="round" stroke-linejoin="round" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
          </svg>
          <!-- ตัวอย่าง Badge แสดงจำนวนของในตะกร้า (ถ้ามี) -->
          <!-- <span class="absolute -top-2 -right-2 bg-red-500 text-white text-xs rounded-full w-5 h-5 flex items-center justify-center">2</span> -->
        </NuxtLink>

        <!-- ปุ่ม Hamburger Menu (แสดงเฉพาะหน้าจอมือถือ) -->
        <button @click="isMobileMenuOpen = !isMobileMenuOpen" class="md:hidden block focus:outline-none">
          <svg xmlns="http://www.w3.org/2000/svg" class="w-7 h-7" fill="none" viewBox="0 0 24 24" stroke="currentColor" stroke-width="2">
            <path v-if="!isMobileMenuOpen" stroke-linecap="round" stroke-linejoin="round" d="M4 6h16M4 12h16M4 18h16" />
            <path v-else stroke-linecap="round" stroke-linejoin="round" d="M6 18L18 6M6 6l12 12" />
          </svg>
        </button>
      </div>
    </header>

    <!-- จอ MOBILE -->
    <!-- เมนู Dropdown สำหรับหน้าจอมือถือ -->
    <transition name="fade">
      <nav v-if="isMobileMenuOpen" class="md:hidden bg-[#ff8fa3] text-white font-bold text-center py-4 space-y-4 shadow-lg">
        <NuxtLink to="/" class="block hover:text-pink-200 transition" @click="isMobileMenuOpen = false">Home</NuxtLink>
        <NuxtLink to="/about" class="block underline decoration-2 underline-offset-4" @click="isMobileMenuOpen = false">About us</NuxtLink>
        <NuxtLink to="/productscategories" class="block hover:text-pink-200 transition" @click="isMobileMenuOpen = false">Product</NuxtLink>
        <NuxtLink to="/faq" class="block hover:text-pink-200 transition" @click="isMobileMenuOpen = false">FAQ</NuxtLink>
        <NuxtLink to="/contact" class="block hover:text-pink-200 transition" @click="isMobileMenuOpen = false">Contact us</NuxtLink>
      </nav>
    </transition>
  </div>
</template>

<style scoped>
/* อนิเมชั่นตอนเปิด/ปิดเมนูมือถือเบาๆ */
.fade-enter-active, .fade-leave-active {
  transition: opacity 0.3s ease, transform 0.3s ease;
}
.fade-enter-from, .fade-leave-to {
  opacity: 0;
  transform: translateY(-10px);
}
 /*ขีนเส้นใต้คำ NavBar*/
.active {
  color: #ffffff !important; /* เปลี่ยนสีตัวอักษรให้เป็นสีหลัก */
  border-bottom: 3px solid #ffffff; /* สร้างขีดเส้นใต้ */
  padding-bottom: 4px; /* เว้นระยะห่างระหว่างตัวอักษรกับเส้น */
  font-weight: 700; /* ทำให้ตัวหนาขึ้นเล็กน้อย (ถ้าต้องการ) */
}
</style>