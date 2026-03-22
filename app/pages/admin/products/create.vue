<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

definePageMeta({ layout: 'admin' })
const router = useRouter()

// ตัวแปรเก็บข้อมูลฟอร์มสินค้า
const form = ref({
  sku: '',
  name: '',
  category_id: '',
  description: '',
  sell_price: 0,
  stock_qty: 0,
  status: 'active'
})

// ข้อมูลจำลองสำหรับ Dropdown หมวดหมู่
const categories = ref([
  { id: 'cat-1', name: 'สติกเกอร์ (Stickers)' },
  { id: 'cat-2', name: 'พวงกุญแจ (Keychains)' },
  { id: 'cat-3', name: 'โปสการ์ด (Postcards)' }
])

const isSubmitting = ref(false)

// ฟังก์ชันบันทึกข้อมูล
const submitProduct = async () => {
  if (!form.value.sku || !form.value.name || !form.value.category_id) {
    alert('กรุณากรอกข้อมูลบังคับให้ครบถ้วน (*)')
    return
  }
  
  isSubmitting.value = true
  
  // จำลองการบันทึกข้อมูลลง Database
  setTimeout(() => {
    console.log('บันทึกข้อมูลสินค้า:', form.value)
    alert('บันทึกสินค้าใหม่เรียบร้อยแล้ว!')
    isSubmitting.value = false
    router.push('/admin/products') // กลับไปหน้ารายการ
  }, 800)
}

const cancel = () => {
  router.push('/admin/products')
}
</script>

<template>
  <div class="p-6 max-w-4xl mx-auto bg-gray-50 min-h-screen">
    
    <div class="mb-8">
      <h1 class="text-3xl font-bold text-gray-800">📦 เพิ่มสินค้าใหม่ (Create Product)</h1>
      <p class="text-gray-500 mt-2">กรอกข้อมูลรายละเอียดสินค้าเพื่อนำไปแสดงหน้าร้าน</p>
    </div>

    <form @submit.prevent="submitProduct" class="bg-white p-8 rounded-2xl shadow-sm border border-gray-200">
      
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">รหัสสินค้า (SKU) <span class="text-red-500">*</span></label>
          <input v-model="form.sku" type="text" placeholder="เช่น PRD-001" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-gray-50 uppercase" />
        </div>
        
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">หมวดหมู่ <span class="text-red-500">*</span></label>
          <select v-model="form.category_id" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-gray-50">
            <option value="" disabled>เลือกหมวดหมู่</option>
            <option v-for="cat in categories" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
          </select>
        </div>
      </div>

      <div class="mb-6">
        <label class="block text-sm font-bold text-gray-700 mb-2">ชื่อสินค้า <span class="text-red-500">*</span></label>
        <input v-model="form.name" type="text" placeholder="ระบุชื่อสินค้า..." required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-gray-50" />
      </div>

      <div class="mb-6">
        <label class="block text-sm font-bold text-gray-700 mb-2">รายละเอียด (Description)</label>
        <textarea v-model="form.description" rows="4" placeholder="อธิบายเกี่ยวกับสินค้า..." class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-gray-50 resize-none"></textarea>
      </div>

      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-8 border-t border-gray-100 pt-6">
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">ราคาขาย (บาท) <span class="text-red-500">*</span></label>
          <input v-model.number="form.sell_price" type="number" min="0" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-gray-50" />
        </div>
        
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">สต๊อกเริ่มต้น (ชิ้น) <span class="text-red-500">*</span></label>
          <input v-model.number="form.stock_qty" type="number" min="0" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-gray-50" />
        </div>
      </div>

      <div class="flex items-center justify-end gap-4 border-t border-gray-100 pt-6">
        <button type="button" @click="cancel" class="px-6 py-2.5 rounded-xl font-bold text-gray-600 bg-gray-100 hover:bg-gray-200 transition">
          ยกเลิก
        </button>
        <button type="submit" :disabled="isSubmitting" class="px-8 py-2.5 rounded-xl font-bold text-white bg-[#ff8fa3] hover:bg-pink-500 shadow-sm transition disabled:opacity-50">
          {{ isSubmitting ? 'กำลังบันทึก...' : 'บันทึกสินค้า' }}
        </button>
      </div>
      
    </form>
  </div>
</template>