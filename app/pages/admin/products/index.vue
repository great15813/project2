<script setup>
import { ref, computed, onMounted } from 'vue'

definePageMeta({
  layout: 'admin',
})

const searchQuery = ref('')
const selectedCategory = ref('all')

// ข้อมูลสินค้า
const products = ref([
  { id: 'PRD-001', name: 'สติกเกอร์น้องกระต่ายพาสเทล', category: 'Stickers', price: 129, stock: 45, image: 'https://via.placeholder.com/150' },
  { id: 'PRD-002', name: 'พวงกุญแจดอกทิวลิป', category: 'Keychains', price: 89, stock: 8, image: 'https://via.placeholder.com/150' },
  { id: 'PRD-003', name: 'โปสการ์ดลายคราฟต์', category: 'Postcards', price: 59, stock: 0, image: 'https://via.placeholder.com/150' },
])
const isLoading = ref(true)

/*
const filteredProducts = computed(() => {
  return products.value.filter(product => {
    const matchSearch = product.name.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
                        product.id.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchCategory = selectedCategory.value === 'all' || product.category === selectedCategory.value
    return matchSearch && matchCategory
  })
})
*/
// 2. เพิ่ม Computed Property เพื่อกรองสินค้าตามคำค้นหา
const filteredProducts = computed(() => {
  // ถ้าช่องค้นหาว่างเปล่า ให้แสดงสินค้าทั้งหมด
  if (!searchQuery.value) return products.value
  
  const lowerCaseQuery = searchQuery.value.toLowerCase()
  
  // ค้นหาจากชื่อสินค้า (name) หรือ รหัสสินค้า (sku)
  return products.value.filter(product => 
    product.name.toLowerCase().includes(lowerCaseQuery) || 
    product.sku.toLowerCase().includes(lowerCaseQuery)
  )
})

// ==========================================
// [ระบบใหม่: จัดการ Popup อัปเดตสต๊อก]
// ==========================================
const isStockModalOpen = ref(false)
const selectedProduct = ref(null)
const newStockValue = ref(0)

// เปิด Popup
const openStockModal = (product) => {
  selectedProduct.value = product
  newStockValue.value = product.stock // ตั้งค่าเริ่มต้นให้เท่ากับสต๊อกปัจจุบัน
  isStockModalOpen.value = true
}

// ปิด Popup
const closeStockModal = () => {
  isStockModalOpen.value = false
  selectedProduct.value = null
  newStockValue.value = 0
}

// ==========================================
// [BACKEND: บันทึกการอัปเดตสต๊อก ลงตาราง STOCK_MOVEMENTS]
// ==========================================
const confirmStockUpdate = async () => {
  if (newStockValue.value < 0) {
    alert('จำนวนสต๊อกติดลบไม่ได้ครับ')
    return
  }

  const oldStock = selectedProduct.value.stock
  const updatedStock = newStockValue.value
  const productId = selectedProduct.value.id
  
  // เช็กว่าสต๊อกไม่มีการเปลี่ยนแปลง ก็ไม่ต้องทำอะไร
  if (oldStock === updatedStock) {
    closeStockModal()
    return
  }

  // คำนวณหาว่าเป็นการ รับเข้า (In) หรือ เบิกออก (Out)
  const difference = updatedStock - oldStock
  const qtyIn = difference > 0 ? difference : 0
  const qtyOut = difference < 0 ? Math.abs(difference) : 0

  // สมมติไอดีของ Admin ที่ล็อกอินอยู่ (ตาราง USERS)
  const adminId = 1 

  try {
    /*
    // โค้ดสำหรับต่อ Supabase ของจริง
    
    // 1. อัปเดตสต๊อกในตาราง PRODUCTS
    await supabase
      .from('PRODUCTS')
      .update({ stock_qty: updatedStock })
      .eq('id', productId)

    // 2. บันทึกประวัติลงตาราง STOCK_MOVEMENTS
    await supabase
      .from('STOCK_MOVEMENTS')
      .insert({
        item_type: 'product',
        movement_type: 'adjustment', // บอกว่าเป็นการปรับปรุงยอดด้วยมือ
        product_id: productId,
        qty_in: qtyIn,
        qty_out: qtyOut,
        balance_after: updatedStock,
        created_by: adminId,
        note: 'Manual stock adjustment by Admin'
      })
    */

    // 👇 โค้ดจำลองการทำงานฝั่งหน้าบ้าน
    selectedProduct.value.stock = updatedStock 
    
    console.log(`[SUPABASE LOG] Insert into STOCK_MOVEMENTS:`, {
      item_type: 'product',
      movement_type: 'adjustment',
      product_id: productId,
      qty_in: qtyIn,
      qty_out: qtyOut,
      balance_after: updatedStock,
      created_by: adminId
    })

    alert(`อัปเดตสต๊อกเรียบร้อยแล้ว!`)
    closeStockModal()

  } catch (error) {
    console.error('อัปเดตสต๊อกล้มเหลว', error)
    alert('เกิดข้อผิดพลาด ไม่สามารถอัปเดตสต๊อกได้')
  }
}

const deleteProduct = (id) => {
  // ฟังก์ชันลบสินค้า (โค้ดเดิม)
  if (window.confirm(`⚠️ แน่ใจหรือไม่ว่าต้องการลบสินค้ารหัส ${id} ?`)) {
    products.value = products.value.filter(p => p.id !== id)
  }
}


</script>

<template>
  <div class="p-4 sm:p-6 lg:p-8 bg-gray-50 min-h-screen relative">
    
    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
      <div>
        <h1 class="text-2xl sm:text-3xl font-bold text-gray-800">Products & Inventory</h1>
        <p class="text-gray-500 mt-1">จัดการรายการสินค้าและสต๊อกคงเหลือ</p>
      </div>
      <!-- UI ช่องค้นหา -->
      <div class="flex flex-col sm:flex-row items-center gap-3 w-full lg:w-auto">
        
        <div class="relative w-full sm:w-72">
          <input 
            v-model="searchQuery" 
            type="text" 
            placeholder="ค้นหาชื่อสินค้า, รหัสสินค้า..." 
            class="w-full pl-10 pr-4 py-2.5 text-sm border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#ff8fa3] focus:border-[#ff8fa3] outline-none transition-all shadow-sm"
          />
          <div class="absolute left-3 top-2.5 text-gray-400 pointer-events-none">
            <svg class="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" />
            </svg>
          </div>
        </div>

        <NuxtLink to="/admin/products/create" class="w-full sm:w-auto bg-[#ff8fa3] hover:bg-pink-400 text-white font-bold py-2.5 px-5 rounded-xl shadow-sm transition whitespace-nowrap">
          + เพิ่มสินค้าใหม่
        </NuxtLink>
      </div>
    </div>
    
    <div class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse min-w-[800px]">
          <thead>
            <tr class="bg-gray-50 text-gray-600 text-sm border-b border-gray-100">
              <th class="py-4 px-6 font-medium">สินค้า</th>
              <th class="py-4 px-6 font-medium">หมวดหมู่</th>
              <th class="py-4 px-6 font-medium">ราคา</th>
              <th class="py-4 px-6 font-medium text-center">สต๊อกคงเหลือ</th>
              <th class="py-4 px-6 font-medium text-center">สถานะ</th>
              <th class="py-4 px-6 font-medium text-right">จัดการ</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-100 text-sm">
            
            <tr v-for="product in filteredProducts" :key="product.id" class="hover:bg-gray-50/50 transition">
              
              <td class="py-3 px-6">
                <div class="flex items-center gap-4">
                  <img :src="product.image" alt="thumbnail" class="w-12 h-12 rounded-lg object-cover border border-gray-200" />
                  <div>
                    <p class="font-bold text-gray-800">{{ product.name }}</p>
                    <p class="text-xs text-gray-400">{{ product.id }}</p>
                  </div>
                </div>
              </td>
              
              <td class="py-3 px-6 text-gray-500">{{ product.category }}</td>
              <td class="py-3 px-6 font-medium text-gray-800">฿{{ product.price }}</td>
              
              <td class="py-3 px-6 text-center">
                <span class="text-lg font-bold" :class="{'text-red-500': product.stock === 0, 'text-gray-800': product.stock > 0}">
                  {{ product.stock }}
                </span>
              </td>

              <td class="py-3 px-6 text-center">
                <span v-if="product.stock > 20" class="bg-green-100 text-green-700 py-1 px-3 rounded-full text-xs font-bold">ปกติ (In Stock)</span>
                <span v-else-if="product.stock > 0" class="bg-yellow-100 text-yellow-700 py-1 px-3 rounded-full text-xs font-bold">ใกล้หมด</span>
                <span v-else class="bg-red-100 text-red-700 py-1 px-3 rounded-full text-xs font-bold">สินค้าหมด</span>
              </td>

              <td class="py-3 px-6 text-right whitespace-nowrap space-x-2">
                <button 
                  @click="openStockModal(product)"
                  class="p-2 text-teal-600 hover:bg-teal-50 border border-teal-200 rounded-lg shadow-sm transition" title="ปรับปรุงสต๊อก"
                >
                  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M20 7l-8-4-8 4m16 0l-8 4m8-4v10l-8 4m0-10L4 7m8 4v10M4 7v10l8 4" /></svg>
                </button>

                <button class="p-2 text-gray-400 hover:text-blue-500 bg-white border border-gray-200 rounded-lg shadow-sm transition" title="แก้ไขสินค้า">
                  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" /></svg>
                </button>
                
                <button @click="deleteProduct(product.id)" class="p-2 text-red-400 hover:text-red-600 hover:bg-red-50 border border-gray-200 rounded-lg shadow-sm transition" title="ลบสินค้า">
                  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 7l-.867 12.142A2 2 0 0116.138 21H7.862a2 2 0 01-1.995-1.858L5 7m5 4v6m4-6v6m1-10V4a1 1 0 00-1-1h-4a1 1 0 00-1 1v3M4 7h16" /></svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="isStockModalOpen" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black bg-opacity-50 backdrop-blur-sm transition-opacity">
      <div class="bg-white rounded-2xl shadow-xl w-full max-w-md overflow-hidden transform transition-all">
        
        <div class="px-6 py-4 border-b border-gray-100 flex justify-between items-center">
          <h3 class="text-lg font-bold text-gray-800">ปรับปรุงสต๊อก (Update Stock)</h3>
          <button @click="closeStockModal" class="text-gray-400 hover:text-gray-600">
            <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>

        <div class="p-6">
          <p class="text-sm text-gray-500 mb-4">รหัสสินค้า: <span class="font-bold text-gray-800">{{ selectedProduct.id }}</span></p>
          <div class="flex items-center gap-4 mb-6 bg-gray-50 p-3 rounded-xl border border-gray-100">
             <img :src="selectedProduct.image" class="w-12 h-12 rounded-md object-cover" />
             <p class="font-medium text-gray-800">{{ selectedProduct.name }}</p>
          </div>

          <div class="grid grid-cols-2 gap-4 items-center text-center mb-6">
             <div>
               <p class="text-xs text-gray-500 mb-1">สต๊อกปัจจุบัน</p>
               <p class="text-2xl font-bold text-gray-400">{{ selectedProduct.stock }}</p>
             </div>
             <div>
               <p class="text-xs text-gray-500 mb-1 text-[#ff8fa3] font-bold">สต๊อกใหม่ที่ต้องการ</p>
               <input 
                 v-model.number="newStockValue" 
                 type="number" 
                 min="0"
                 class="w-full text-center text-2xl font-bold text-gray-800 border-2 border-gray-200 rounded-xl py-2 focus:ring-0 focus:border-[#ff8fa3] outline-none"
               />
             </div>
          </div>
          
          <div class="flex gap-3">
            <button @click="closeStockModal" class="flex-1 py-3 bg-gray-100 text-gray-600 font-bold rounded-xl hover:bg-gray-200 transition">
              ยกเลิก
            </button>
            <button @click="confirmStockUpdate" class="flex-1 py-3 bg-[#ff8fa3] text-white font-bold rounded-xl hover:bg-pink-400 shadow-sm transition">
              บันทึกข้อมูล
            </button>
          </div>

        </div>
      </div>
    </div>

  </div>
</template>