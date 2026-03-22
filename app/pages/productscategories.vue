<script setup>
import { ref, onMounted } from 'vue'
// [BACKEND: ส่วนที่ 1 - เรียกใช้ Composable]
// ในโปรเจกต์ของคุณมีโฟลเดอร์ composables/useProducts.ts อยู่ 
// คุณสามารถ import ฟังก์ชันสำหรับดึงหมวดหมู่จาก Supabase มาใช้ตรงนี้ได้ครับ
// ตัวอย่าง: const { fetchCategories } = useProducts()

// [BACKEND: ส่วนที่ 2 - สร้างตัวแปรรับข้อมูล]
// ตัวแปรนี้จะรอรับข้อมูลที่ดึงมาจาก Database (ตอนนี้ผมใส่ข้อมูลจำลองไว้ให้ดูโครงสร้างก่อน)
const categories = ref([
  { id: 1, name: 'โมบาย', image_url: 'https://via.placeholder.com/600x800?text=Mobile', slug: 'mobiles' },
  { id: 2, name: 'พวงกุญแจ', image_url: 'https://via.placeholder.com/600x800?text=Keychain', slug: 'keychains' },
  { id: 3, name: 'ยางรัดผม', image_url: 'https://via.placeholder.com/600x800?text=Hairband', slug: 'hairbands' },
])

// [BACKEND: ส่วนที่ 3 - ดึงข้อมูลตอนเปิดหน้าเว็บ]
// เมื่อหน้าเว็บโหลด (onMounted) เราจะสั่งให้ดึงข้อมูลของจริงจาก Backend มาทับข้อมูลจำลอง
/*
onMounted(async () => {
  try {
    // สมมติว่าดึงข้อมูลจาก Supabase ผ่าน useProducts()
    // const data = await fetchCategories()
    // categories.value = data
  } catch (error) {
    console.error('ดึงข้อมูลหมวดหมู่ล้มเหลว:', error)
  }
})
*/
</script>

<template>
  
  <div class="min-h-screen bg-[#fff5f7] py-12 px-4 sm:px-6 lg:px-8">
    
    <div class="max-w-7xl mx-auto text-center mb-10">
      <h1 class="text-3xl md:text-4xl font-bold text-gray-800">เลือกชมตามหมวดหมู่</h1>
      <p class="text-gray-500 mt-3 text-sm md:text-base">สินค้าแฮนด์เมดน่ารักๆ รอคุณอยู่</p>
    </div>

    <div class="max-w-5xl mx-auto grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 gap-6 md:gap-8">
      <!-- เชื่อมหน้ารายการproduct -->
      <NuxtLink 
        v-for="category in categories" 
        :key="category.id"
        :to="`/productsList?category=${category.slug}`" 
        class="group flex flex-col items-center bg-white rounded-[2rem] p-4 shadow-sm hover:shadow-xl transition-all duration-300 cursor-pointer"
      >
        <div class="w-full aspect-[4/5] overflow-hidden rounded-[1.5rem] bg-pink-50 relative">
          <img 
            :src="category.image_url" 
            :alt="category.name" 
            class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-500 ease-in-out" 
          />
          
          <div class="absolute inset-0 bg-black/10 opacity-0 group-hover:opacity-100 transition-opacity duration-300"></div>
        </div>
        
        <h3 class="mt-6 mb-2 text-xl md:text-2xl font-bold text-gray-700 group-hover:text-[#ff8fa3] transition-colors">
          {{ category.name }}
        </h3>
      </NuxtLink>

    </div>

  </div>
</template>