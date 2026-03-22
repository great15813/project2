<script setup>
import { ref } from 'vue'

// ตัวแปรจำลองสำหรับจัดการสไลด์ภาพ Hero Banner
const currentSlide = ref(0)
const heroImages = [
  '/Home/Home1.png', // ใส่ path รูปแบนเนอร์ของคุณที่นี่
  '/Home/Home2.png',
  '/Home/Home3.png',
  '/Home/Home4.png'
]

// ฟังก์ชันเปลี่ยนสไลด์เมื่อกดจุด (Dots)
const setSlide = (index) => {
  currentSlide.value = index
}

// TODO: ในอนาคตสามารถดึงข้อมูลสินค้าจริงจาก useProducts() ได้
// const { products } = useProducts()
</script>

<template>
  <div class="bg-white min-h-screen pb-12">
    
    <!-- 1. HERO BANNER SECTION -->
    <section class="w-full flex flex-col items-center pt-14 md:pt-18 px-4 md:px-8">
      
      <!-- กรอบรูปภาพ -->
      <div class="relative w-full max-w-6xl rounded-2xl md:rounded-[2rem] overflow-hidden shadow-md group">
        <!-- ภาพแบนเนอร์ -->
        <!-- [Responsive]: มือถือใช้สัดส่วน 4:3, จอคอมใช้ 21:9 และใช้ object-cover เพื่อไม่ให้ภาพผิดเพี้ยน -->
        <img 
          :src="heroImages[currentSlide]" 
          alt="Featured Craft Products" 
          class="w-full aspect-[4/3] md:aspect-[21/9] object-cover transition-all duration-500 ease-in-out"
        />

        <!-- ข้อความป้ายกำกับ (ถ้าอยากให้มีข้อความทับบนรูป ลบ comment ออกได้ครับ) -->
        <!-- 
        <div class="absolute inset-0 bg-black/20 flex items-center justify-center">
          <h1 class="text-white text-3xl md:text-5xl font-bold drop-shadow-lg">Welcome to Our Shop</h1>
        </div> 
        -->
      </div>

      <!-- จุดนำทาง (Dots Indicator) -->
      <div class="flex space-x-3 md:space-x-4 mt-6 md:mt-8">
        <button 
          v-for="(img, index) in heroImages" 
          :key="index"
          @click="setSlide(index)"
          :class="[
            'w-3 h-3 md:w-4 md:h-4 rounded-full transition-all duration-300 focus:outline-none',
            currentSlide === index ? 'bg-[#ff8fa3] scale-125' : 'bg-gray-300 hover:bg-gray-400'
          ]"
          :aria-label="`Go to slide ${index + 1}`"
        ></button>
      </div>
    </section>

    <!-- 2. FEATURED PRODUCTS SECTION (ส่วนเสริม: สินค้าแนะนำ) -->
    <section class="w-full max-w-6xl mx-auto mt-16 px-4 md:px-8">
      <div class="flex items-center justify-between mb-8">
        <h2 class="text-2xl md:text-3xl font-bold text-gray-800">New Arrivals ✨</h2>
        <NuxtLink to="/products" class="text-[#ff8fa3] font-semibold hover:underline">View All</NuxtLink>
      </div>

      <!-- Product Grid -->
      <!-- [Responsive]: มือถือแสดง 2 คอลัมน์ (grid-cols-2), จอใหญ่แสดง 4 คอลัมน์ (md:grid-cols-4) -->
      <div class="grid grid-cols-2 md:grid-cols-4 gap-4 md:gap-6">
        
        <!-- การ์ดสินค้าจำลอง (วนลูปสร้าง 4 ชิ้น) -->
        <div v-for="n in 4" :key="n" class="bg-pink-50 rounded-xl overflow-hidden shadow-sm hover:shadow-md transition duration-300 group cursor-pointer">
          <!-- รูปสินค้า -->
          <div class="w-full aspect-square overflow-hidden bg-white">
            <img 
              src="https://via.placeholder.com/400?text=Product+Image" 
              alt="Product" 
              class="w-full h-full object-cover group-hover:scale-110 transition duration-500"
            />
          </div>
          <!-- รายละเอียดสินค้า -->
          <div class="p-4 text-center md:text-left">
            <h3 class="text-gray-700 font-semibold truncate">Cute Tulip Keychain {{ n }}</h3>
            <p class="text-[#ff8fa3] font-bold mt-1">฿150.00</p>
          </div>
        </div>

      </div>
    </section>

  </div>
</template>