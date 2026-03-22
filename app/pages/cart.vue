<script setup>
import { ref, computed } from 'vue'

// [BACKEND: 1. ข้อมูลตะกร้าสินค้า]
const cartItems = ref([
  { id: 1, name: 'สติกเกอร์น้องกระต่ายพาสเทล น่ารักมากก', price: 129, quantity: 1, image: 'https://via.placeholder.com/150', selected: true },
  { id: 2, name: 'พวงกุญแจดอกทิวลิป', price: 89, quantity: 2, image: 'https://via.placeholder.com/150', selected: true },
])

// [BACKEND: 2. ข้อมูลที่อยู่จัดส่ง]
const shippingAddress = ref({
  name: '',
  phone: '',
  address: ''
})

// [BACKEND: 3. โค้ดส่วนลด]
const discountCode = ref('')
const discountAmount = ref(0) // จำนวนเงินที่ลด

// คำนวณยอดต่างๆ (Reactive)
const selectAll = computed({
  get: () => cartItems.value.length > 0 && cartItems.value.every(item => item.selected),
  set: (val) => cartItems.value.forEach(item => item.selected = val)
})

const subTotal = computed(() => {
  return cartItems.value
    .filter(item => item.selected)
    .reduce((sum, item) => sum + (item.price * item.quantity), 0)
})

const shippingFee = computed(() => subTotal.value > 0 ? 50 : 0) // สมมติค่าส่ง 50 บาท
const netTotal = computed(() => subTotal.value + shippingFee.value - discountAmount.value)

// ฟังก์ชันจัดการตะกร้า
const increaseQty = (item) => item.quantity++
const decreaseQty = (item) => { if (item.quantity > 1) item.quantity-- }
const removeItem = (id) => {
  // [BACKEND: ยิง API ลบสินค้าออกจากตะกร้า]
  cartItems.value = cartItems.value.filter(item => item.id !== id)
}

// ฟังก์ชันใช้โค้ดส่วนลด
const applyDiscount = () => {
  // [BACKEND: ส่งโค้ดไปเช็กกับ Server]
  if (discountCode.value === 'NEWUSER') {
    discountAmount.value = 30 // สมมติลด 30 บาท
    alert('ใช้โค้ดส่วนลดสำเร็จ! ลดไป 30 บาท')
  } else {
    alert('โค้ดส่วนลดไม่ถูกต้องหรือหมดอายุ')
    discountAmount.value = 0
  }
}

// ฟังก์ชันยืนยันการสั่งซื้อ
const handleCheckout = () => {
  if (!shippingAddress.value.name || !shippingAddress.value.phone || !shippingAddress.value.address) {
    alert('กรุณากรอกที่อยู่จัดส่งให้ครบถ้วนครับ')
    return
  }
  if (subTotal.value === 0) {
    alert('กรุณาเลือกสินค้าอย่างน้อย 1 ชิ้น')
    return
  }
  
  // [BACKEND: รวบรวมข้อมูลส่งไปสร้าง Order ใหม่]
  console.log('ข้อมูลสั่งซื้อ:', {
    items: cartItems.value.filter(i => i.selected),
    address: shippingAddress.value,
    discount: discountAmount.value,
    total: netTotal.value
  })
  alert('สั่งซื้อสำเร็จ! กำลังพาท่านไปหน้าชำระเงิน...')
  // router.push('/payment')
}
</script>

<template>
  <div class="min-h-screen bg-[#fff5f7] py-8 px-4 sm:px-6 lg:px-8">
    <div class="max-w-7xl mx-auto">
      
      <h1 class="text-2xl md:text-3xl font-bold text-gray-800 mb-8">ตะกร้าสินค้าของคุณ</h1>

      <div class="grid grid-cols-1 lg:grid-cols-3 gap-8 items-start">
        
        <div class="lg:col-span-2 space-y-8">
          
          <div class="bg-white rounded-[2rem] p-6 shadow-sm">
            <div class="flex items-center justify-between border-b border-gray-100 pb-4 mb-4">
              <label class="flex items-center gap-3 cursor-pointer">
                <input type="checkbox" v-model="selectAll" class="w-5 h-5 accent-[#ff8fa3] rounded" />
                <span class="font-medium text-gray-700">เลือกทั้งหมด</span>
              </label>
            </div>

            <div v-if="cartItems.length === 0" class="text-center py-10 text-gray-400">
              ไม่มีสินค้าในตะกร้า
            </div>

            <div class="space-y-4">
              <div 
                v-for="item in cartItems" 
                :key="item.id"
                class="flex flex-col sm:flex-row items-start sm:items-center gap-4 bg-gray-50 p-4 rounded-2xl relative group"
              >
                <div class="flex items-center gap-4 w-full sm:w-auto">
                  <input type="checkbox" v-model="item.selected" class="w-5 h-5 accent-[#ff8fa3] rounded flex-shrink-0" />
                  <img :src="item.image" :alt="item.name" class="w-20 h-20 rounded-xl object-cover border border-gray-200" />
                </div>

                <div class="flex-1 w-full">
                  <h3 class="font-medium text-gray-800 line-clamp-2 pr-8">{{ item.name }}</h3>
                  <p class="text-[#ff8fa3] font-bold mt-1">฿{{ item.price }}</p>
                </div>

                <div class="flex items-center justify-between sm:justify-end w-full sm:w-auto gap-4 mt-2 sm:mt-0">
                  
                  <button @click="removeItem(item.id)" class="text-gray-400 hover:text-red-500 transition sm:absolute sm:top-4 sm:right-4">
                    <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                      <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" />
                    </svg>
                  </button>

                  <div class="flex items-center border-2 border-pink-100 bg-white rounded-full px-2 h-10 w-28">
                    <button @click="decreaseQty(item)" class="text-xl text-[#ff8fa3] w-8 h-full flex items-center justify-center font-bold focus:outline-none" :disabled="item.quantity <= 1">-</button>
                    <span class="text-base font-bold text-gray-800 flex-1 text-center select-none">{{ item.quantity }}</span>
                    <button @click="increaseQty(item)" class="text-xl text-[#ff8fa3] w-8 h-full flex items-center justify-center font-bold focus:outline-none">+</button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div class="bg-white rounded-[2rem] p-6 shadow-sm">
            <h2 class="text-xl font-bold text-gray-800 mb-4 flex items-center gap-2">
              <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6 text-[#ff8fa3]" fill="none" viewBox="0 0 24 24" stroke="currentColor">
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M17.657 16.657L13.414 20.9a1.998 1.998 0 01-2.827 0l-4.244-4.243a8 8 0 1111.314 0z" />
                <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 11a3 3 0 11-6 0 3 3 0 016 0z" />
              </svg>
              ที่อยู่จัดส่ง
            </h2>
            <div class="space-y-4">
              <div class="grid grid-cols-1 md:grid-cols-2 gap-4">
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">ชื่อ-นามสกุล</label>
                  <input v-model="shippingAddress.name" type="text" placeholder="ชื่อผู้รับ" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-2.5 focus:ring-2 focus:ring-[#ff8fa3] outline-none" />
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-1">เบอร์โทรศัพท์</label>


                  
                  <input v-model="shippingAddress.phone" type="tel" placeholder="08X-XXX-XXXX" class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-2.5 focus:ring-2 focus:ring-[#ff8fa3] outline-none" />
                </div>
              </div>
              <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">ที่อยู่จัดส่ง (บ้านเลขที่, ซอย, ถนน, ตำบล, อำเภอ, จังหวัด, รหัสไปรษณีย์)</label>
                <textarea v-model="shippingAddress.address" rows="3" placeholder="กรอกที่อยู่สำหรับจัดส่งให้ชัดเจน..." class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none resize-none"></textarea>
              </div>
            </div>
          </div>

        </div>

        <div class="lg:col-span-1">
          <div class="bg-white rounded-[2rem] p-6 shadow-sm sticky top-24 space-y-6">
            
            <h2 class="text-xl font-bold text-gray-800">สรุปคำสั่งซื้อ (Order Detail)</h2>

            <div>
                <label class="block text-sm font-medium text-gray-700 mb-2">โค้ดส่วนลด (ถ้ามี)</label>
                <div class="relative flex items-center">
                    <input 
                    v-model="discountCode" 
                    type="text" 
                    placeholder="กรอกโค้ด..." 
                    class="w-full bg-gray-50 border border-gray-200 rounded-xl pl-4 pr-24 py-3 uppercase focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all" 
                    />
                    <button 
                    @click="applyDiscount" 
                    class="absolute right-1 top-1 bottom-1 bg-gray-800 text-white font-medium px-4 rounded-lg hover:bg-gray-700 transition"
                    >
                    Apply
                    </button>
                </div>
                </div>

            <div class="space-y-3 border-t border-b border-gray-100 py-4 text-gray-600">
              <div class="flex justify-between">
                <span>ยอดรวม (Total)</span>
                <span class="font-medium text-gray-800">฿{{ subTotal }}</span>
              </div>
              <div class="flex justify-between">
                <span>ค่าจัดส่ง (Shipping)</span>
                <span class="font-medium text-gray-800">฿{{ shippingFee }}</span>
              </div>
              <div class="flex justify-between text-green-500" v-if="discountAmount > 0">
                <span>ส่วนลด (Discount)</span>
                <span class="font-medium">-฿{{ discountAmount }}</span>
              </div>
            </div>

            <div class="flex justify-between items-center">
              <span class="text-lg font-bold text-gray-800">ยอดสุทธิ (Net Total)</span>
              <span class="text-2xl font-bold text-[#ff8fa3]">฿{{ netTotal }}</span>
            </div>

            <button 
              @click="handleCheckout"
              class="w-full bg-[#ff8fa3] text-white text-lg font-bold py-4 rounded-xl hover:bg-pink-400 hover:shadow-md transition-all mt-4"
            >
              ชำระเงิน (Checkout)
            </button>

          </div>
        </div>

      </div>

    </div>
  </div>
</template>