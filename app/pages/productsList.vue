<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRoute } from 'vue-router'

// ใช้สำหรับเช็คว่าผู้ใช้กดหมวดหมู่ไหนมา (เช่น /products?category=keychains)
const route = useRoute()
const currentCategory = computed(() => route.query.category || 'ทั้งหมด')

// [BACKEND: ส่วนที่ 1 - เรียกใช้ Composables]
// ดึงข้อมูลสินค้าจาก Backend และระบบตะกร้า
// const { fetchProducts } = useProducts()
// const { addToCart } = useCart()

// [BACKEND: ส่วนที่ 2 - ข้อมูลสินค้าจำลอง] 
//กำหนดรายละเอียดสินค้าวิ่งไปยัง product/[id].vue เพื่อเข้าหน้ารายละเอียดสินค้า
const products = ref([
  { id: 1, name: 'ยางรัดผม + ตุ๊กตาส้ม', price: 50, image: 'https://via.placeholder.com/400x400?text=Hairband', category: 'hairbands' },
  { id: 2, name: 'พวงกุญแจดอกทิวลิป', price: 89, image: 'https://via.placeholder.com/400x400?text=Keychain', category: 'keychains' },
  { id: 3, name: 'โมบายห้อยกระเป๋า', price: 120, image: 'https://via.placeholder.com/400x400?text=Mobile', category: 'mobiles' },
  { id: 4, name: 'ที่คั่นหนังสือไหมพรม', price: 35, image: 'https://via.placeholder.com/400x400?text=Bookmark', category: 'others' },
  { id: 5, name: 'ยางรัดผม โบว์ชมพู', price: 45, image: 'https://via.placeholder.com/400x400?text=Hairband2', category: 'hairbands' },
])

// ฟังก์ชันจำลองการหยิบใส่ตะกร้า
const handleAddToCart = (product) => {
  // [BACKEND: ส่วนที่ 3 - ส่งข้อมูลเข้าตะกร้า]
  // addToCart(product)
  alert(`เพิ่ม "${product.name}" ลงตะกร้าแล้ว!`)
}

/*
onMounted(async () => {
  // โค้ดดึงข้อมูลสินค้าจาก Backend ของจริง
  // const data = await fetchProducts(currentCategory.value)
  // products.value = data
})
*/
</script>

<template>
  <div class="min-h-screen bg-[#fff5f7] py-8 px-4 sm:px-6 lg:px-8">
    
    <div class="max-w-7xl mx-auto">
      
      <nav class="flex items-center space-x-2 text-sm md:text-base text-gray-500 mb-8 overflow-x-auto whitespace-nowrap pb-2">
        <NuxtLink to="/" class="hover:text-[#ff8fa3] font-medium transition">Home</NuxtLink>
        <span>/</span>
        <NuxtLink to="/categories" class="hover:text-[#ff8fa3] font-medium transition">Products</NuxtLink>
        <span>/</span>
        <span class="text-[#ff8fa3] font-bold">{{ currentCategory }}</span>
      </nav>

      <div class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4 md:gap-6">
        
        <div 
          v-for="product in products" 
          :key="product.id" 
          class="bg-[#e6f4f1] rounded-2xl p-3 md:p-4 flex flex-col h-full shadow-sm hover:shadow-md transition-shadow relative group"
        >
          <NuxtLink :to="`/product/${product.id}`" class="block rounded-xl overflow-hidden bg-white aspect-square mb-3 relative">
            <img 
              :src="product.image" 
              :alt="product.name" 
              class="w-full h-full object-cover group-hover:scale-110 transition-transform duration-500" 
            />
          </NuxtLink>

          <NuxtLink :to="`/product/${product.id}`" class="flex-grow">
            <h3 class="text-sm md:text-base font-medium text-gray-800 line-clamp-2 hover:text-[#ff8fa3] transition">
              {{ product.name }}
            </h3>
          </NuxtLink>

          <div class="flex justify-between items-end mt-2 pt-2 border-t border-teal-100">
            <p class="font-bold text-gray-900 text-lg md:text-xl">{{ product.price }}.-</p>
            
            <button 
              @click.prevent="handleAddToCart(product)"
              class="bg-white text-[#ff8fa3] p-2 md:p-2.5 rounded-full shadow-sm hover:bg-[#ff8fa3] hover:text-white transition-colors focus:outline-none focus:ring-2 focus:ring-pink-300"
              aria-label="Add to cart"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 md:h-6 md:w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
              </svg>
            </button>
          </div>
        </div>

      </div>

    </div>
  </div>
</template>