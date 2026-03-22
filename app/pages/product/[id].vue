<script setup>
import { ref, onMounted } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
// ดึง ID ของสินค้าจาก URL (เช่น /product/1 -> productId จะเป็น 1)
const productId = route.params.id

// [BACKEND: ส่วนที่ 1 - เตรียมฟังก์ชัน]
// const { fetchProductById } = useProducts()
// const { addToCart } = useCart()

// ตัวแปรเก็บข้อมูลสินค้าจำลอง (รอรับจาก Backend)
const product = ref({
  id: productId,
  name: 'สติกเกอร์น้องกระต่ายพาสเทล น่ารักมากก',
  price: 129,
  description: 'สติกเกอร์ PVC กันน้ำ ลายคราฟต์วาดมือ เคลือบโฮโลแกรมวิบวับ เหมาะสำหรับติดเคสโทรศัพท์ ไอแพด หรือสมุดโน้ต ขนาด 10x10 cm.',
  images: [
    'https://via.placeholder.com/800x800?text=Main+Image',
    'https://via.placeholder.com/800x800?text=Thumb+1',
    'https://via.placeholder.com/800x800?text=Thumb+2'
  ],
  category: 'stickers'
})

// ตัวแปรจัดการ UI
const currentImage = ref(product.value.images[0]) // รูปภาพที่แสดงอยู่ตอนแรก
const quantity = ref(1) // จำนวนสินค้าเริ่มต้น

// ฟังก์ชันเพิ่ม/ลด จำนวนสินค้า
const increaseQuantity = () => {
  quantity.value++
}
const decreaseQuantity = () => {
  if (quantity.value > 1) {
    quantity.value--
  }
}

// ฟังก์ชันกดใส่ตะกร้า
const handleAddToCart = () => {
  // [BACKEND: ส่วนที่ 2 - โยนข้อมูลเข้า Store ตะกร้า]
  /*
  addToCart({
    product: product.value,
    quantity: quantity.value
  })
  */
  alert(`เพิ่ม "${product.value.name}" จำนวน ${quantity.value} ชิ้น ลงในตะกร้าแล้ว! 🛒`)
}

// [BACKEND: ส่วนที่ 3 - ดึงข้อมูลเมื่อเปิดหน้า]
/*
onMounted(async () => {
  try {
    const data = await fetchProductById(productId)
    if (data) {
      product.value = data
      currentImage.value = data.images[0]
    }
  } catch (error) {
    console.error('ไม่พบสินค้า:', error)
  }
})
*/
</script>

<template>
  <div class="min-h-screen bg-[#fff5f7] py-8 px-4 sm:px-6 lg:px-8">
    
    <div class="max-w-6xl mx-auto bg-white rounded-[2rem] shadow-sm p-4 md:p-8 lg:p-10">
      
      <div class="flex flex-col md:flex-row gap-8 lg:gap-12">
        
        <div class="w-full md:w-1/2 flex flex-col gap-4">
          <div class="w-full aspect-square rounded-2xl overflow-hidden bg-pink-50 border border-gray-100 relative">
            <img 
              :src="currentImage" 
              :alt="product.name" 
              class="w-full h-full object-cover transition-opacity duration-300" 
            />
          </div>

          <div class="flex gap-3 overflow-x-auto pb-2 custom-scrollbar">
            <button 
              v-for="(img, index) in product.images" 
              :key="index"
              @click="currentImage = img"
              :class="[
                'flex-shrink-0 w-20 h-20 md:w-24 md:h-24 rounded-xl overflow-hidden border-2 transition-all focus:outline-none',
                currentImage === img ? 'border-[#ff8fa3] opacity-100' : 'border-transparent opacity-60 hover:opacity-100'
              ]"
            >
              <img :src="img" alt="thumbnail" class="w-full h-full object-cover" />
            </button>
          </div>
        </div>

        <div class="w-full md:w-1/2 flex flex-col">
          
          <div class="text-sm text-gray-400 mb-3">
            <NuxtLink to="/products" class="hover:text-[#ff8fa3]">Products</NuxtLink> 
            <span class="mx-1">></span> 
            <span class="capitalize">{{ product.category }}</span>
          </div>

          <h1 class="text-2xl md:text-3xl lg:text-4xl font-bold text-gray-800 leading-tight">
            {{ product.name }}
          </h1>
          <p class="text-3xl md:text-4xl font-bold text-[#ff8fa3] mt-4 mb-6">
            ฿{{ product.price }}
          </p>

          <div class="prose prose-pink text-gray-600 mb-8 border-y border-gray-100 py-6">
            <p>{{ product.description }}</p>
          </div>

          <div class="mt-auto flex flex-col sm:flex-row gap-4">
            
            <div class="flex items-center justify-between border-2 border-pink-100 bg-pink-50 rounded-full px-4 h-14 w-full sm:w-36">
              <button 
                @click="decreaseQuantity" 
                class="text-2xl text-[#ff8fa3] hover:text-pink-600 focus:outline-none w-8 h-8 flex items-center justify-center font-bold"
                :disabled="quantity <= 1"
                :class="{ 'opacity-50 cursor-not-allowed': quantity <= 1 }"
              >
                -
              </button>
              <span class="text-xl font-bold text-gray-800 w-10 text-center select-none">{{ quantity }}</span>
              <button 
                @click="increaseQuantity" 
                class="text-2xl text-[#ff8fa3] hover:text-pink-600 focus:outline-none w-8 h-8 flex items-center justify-center font-bold"
              >
                +
              </button>
            </div>

            <button 
              @click="handleAddToCart"
              class="w-full flex-1 bg-[#ff8fa3] text-white text-lg font-bold rounded-full h-14 hover:bg-pink-400 hover:shadow-lg transition-all duration-300 flex items-center justify-center gap-2"
            >
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 3h2l.4 2M7 13h10l4-8H5.4M7 13L5.4 5M7 13l-2.293 2.293c-.63.63-.184 1.707.707 1.707H17m0 0a2 2 0 100 4 2 2 0 000-4zm-8 2a2 2 0 11-4 0 2 2 0 014 0z" />
              </svg>
              ใส่ตะกร้าเลย
            </button>

          </div>
        </div>

      </div>
    </div>
  </div>
</template>

<style scoped>
/* ซ่อน Scrollbar ที่น่าเกลียด แต่ยังคง Scroll ได้สำหรับรูป Thumbnails */
.custom-scrollbar::-webkit-scrollbar {
  height: 6px;
}
.custom-scrollbar::-webkit-scrollbar-track {
  background: transparent;
}
.custom-scrollbar::-webkit-scrollbar-thumb {
  background-color: #ffd6df;
  border-radius: 10px;
}
</style>