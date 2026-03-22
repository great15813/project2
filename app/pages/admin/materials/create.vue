<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

definePageMeta({ layout: 'admin' })
const router = useRouter()

// ตัวแปรเก็บข้อมูลฟอร์มวัตถุดิบ
const form = ref({
  code: '',
  name: '',
  category_id: '',
  base_unit: '', // เช่น หลา, กิโลกรัม, ชิ้น
  qty_on_hand: 0,
  reorder_level: 10,
  last_cost_per_unit: 0
})

const categories = ref([
  { id: 'mcat-1', name: 'ผ้า (Fabric)' },
  { id: 'mcat-2', name: 'อะไหล่ (Hardware)' },
  { id: 'mcat-3', name: 'บรรจุภัณฑ์ (Packaging)' }
])

const isSubmitting = ref(false)

const submitMaterial = async () => {
  if (!form.value.code || !form.value.name || !form.value.base_unit) {
    alert('กรุณากรอกข้อมูลบังคับให้ครบถ้วน (*)')
    return
  }
  
  isSubmitting.value = true
  
  // จำลองการบันทึกข้อมูล
  setTimeout(() => {
    console.log('บันทึกข้อมูลวัตถุดิบ:', form.value)
    alert('เพิ่มวัตถุดิบใหม่เรียบร้อยแล้ว!')
    isSubmitting.value = false
    router.push('/admin/materials')
  }, 800)
}

const cancel = () => {
  router.push('/admin/materials')
}
</script>

<template>
  <div class="p-6 max-w-4xl mx-auto bg-gray-50 min-h-screen">
    
    <div class="mb-8">
      <h1 class="text-3xl font-bold text-gray-800">🧵 เพิ่มวัตถุดิบใหม่ (Create Material)</h1>
      <p class="text-gray-500 mt-2">เพิ่มรายการวัตถุดิบที่ใช้สำหรับการผลิตสินค้า</p>
    </div>

    <form @submit.prevent="submitMaterial" class="bg-white p-8 rounded-2xl shadow-sm border border-gray-200">
      
      <div class="grid grid-cols-1 md:grid-cols-2 gap-6 mb-6">
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">รหัสวัตถุดิบ (Code) <span class="text-red-500">*</span></label>
          <input v-model="form.code" type="text" placeholder="เช่น MAT-001" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400 outline-none bg-gray-50 uppercase" />
        </div>
        
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">หมวดหมู่</label>
          <select v-model="form.category_id" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400 outline-none bg-gray-50">
            <option value="" disabled>เลือกหมวดหมู่</option>
            <option v-for="cat in categories" :key="cat.id" :value="cat.id">{{ cat.name }}</option>
          </select>
        </div>
      </div>

      <div class="mb-8 border-b border-gray-100 pb-8">
        <label class="block text-sm font-bold text-gray-700 mb-2">ชื่อวัตถุดิบ <span class="text-red-500">*</span></label>
        <input v-model="form.name" type="text" placeholder="เช่น ผ้าคอตตอน 100% สีขาว" required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400 outline-none bg-gray-50" />
      </div>

      <h3 class="text-lg font-bold text-gray-800 mb-4">ข้อมูลคลังสินค้า (Inventory Config)</h3>
      <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">หน่วยนับ <span class="text-red-500">*</span></label>
          <input v-model="form.base_unit" type="text" placeholder="เช่น หลา, กก." required class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400 outline-none bg-gray-50" />
        </div>
        
        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">จุดแจ้งเตือน (Reorder Level)</label>
          <input v-model.number="form.reorder_level" type="number" min="0" placeholder="ตัวเลขเตือนสั่งของ" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400 outline-none bg-gray-50" />
        </div>

        <div>
          <label class="block text-sm font-bold text-gray-700 mb-2">ต้นทุนล่าสุด/หน่วย (บาท)</label>
          <input v-model.number="form.last_cost_per_unit" type="number" min="0" step="0.01" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-blue-400 outline-none bg-gray-50" />
        </div>
      </div>

      <div class="flex items-center justify-end gap-4 border-t border-gray-100 pt-6">
        <button type="button" @click="cancel" class="px-6 py-2.5 rounded-xl font-bold text-gray-600 bg-gray-100 hover:bg-gray-200 transition">
          ยกเลิก
        </button>
        <button type="submit" :disabled="isSubmitting" class="px-8 py-2.5 rounded-xl font-bold text-white bg-blue-600 hover:bg-blue-500 shadow-sm transition disabled:opacity-50">
          {{ isSubmitting ? 'กำลังบันทึก...' : 'บันทึกวัตถุดิบ' }}
        </button>
      </div>
      
    </form>
  </div>
</template>