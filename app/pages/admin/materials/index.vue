<script setup>
import { ref, computed } from 'vue'

definePageMeta({
  layout: 'admin',
})

const searchQuery = ref('')
const selectedCategory = ref('all')

// ==========================================
// [BACKEND: 1. ข้อมูลจำลองจากตาราง MATERIALS]
// ==========================================
const materials = ref([
  { id: 'MAT-001', name: 'ผ้าคอตตอน 100% สีชมพูพาสเทล', category: 'Fabric', qty_on_hand: 45, reorder_level: 10, unit: 'หลา', cost: 120 },
  { id: 'MAT-002', name: 'ใยสังเคราะห์ (ยัดตุ๊กตา)', category: 'Filling', qty_on_hand: 5, reorder_level: 10, unit: 'กิโลกรัม', cost: 80 },
  { id: 'MAT-003', name: 'ห่วงพวงกุญแจ สีเงิน', category: 'Hardware', qty_on_hand: 500, reorder_level: 100, unit: 'ชิ้น', cost: 2 },
  { id: 'MAT-004', name: 'ด้ายเย็บผ้า สีขาว', category: 'Thread', qty_on_hand: 0, reorder_level: 5, unit: 'หลอด', cost: 15 },
  { id: 'MAT-005', name: 'กระดุมไม้ตกแต่ง 15mm', category: 'Accessories', qty_on_hand: 120, reorder_level: 50, unit: 'เม็ด', cost: 3 },
])

const filteredMaterials = computed(() => {
  return materials.value.filter(mat => {
    const matchSearch = mat.name.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
                        mat.id.toLowerCase().includes(searchQuery.value.toLowerCase())
    const matchCategory = selectedCategory.value === 'all' || mat.category === selectedCategory.value
    return matchSearch && matchCategory
  })
})

// ==========================================
// [ระบบ Popup: จัดการเบิก/รับเข้าวัตถุดิบแบบเหมา]
// ==========================================
const isModalOpen = ref(false)
const selectedMat = ref(null)
const actionType = ref('in') // 'in' = รับเข้า, 'out' = เบิกออก
const actionQty = ref(0)
const actionNote = ref('')

const openActionModal = (mat, type) => {
  selectedMat.value = mat
  actionType.value = type
  actionQty.value = 0
  actionNote.value = type === 'out' ? 'เบิกไปผลิต...' : 'รับเข้าจาก Supplier...'
  isModalOpen.value = true
}

const closeModal = () => {
  isModalOpen.value = false
  selectedMat.value = null
  actionQty.value = 0
  actionNote.value = ''
}

// ==========================================
// [BACKEND: 2. บันทึกสต๊อกและเก็บ Log]
// ==========================================
const confirmAction = async () => {
  if (actionQty.value <= 0) {
    alert('กรุณากรอกจำนวนให้มากกว่า 0')
    return
  }

  const mat = selectedMat.value
  const oldQty = mat.qty_on_hand
  let newQty = oldQty

  // คำนวณยอดใหม่
  if (actionType.value === 'in') {
    newQty = oldQty + actionQty.value
  } else {
    if (oldQty - actionQty.value < 0) {
      alert('จำนวนเบิกออกมากกว่าสต๊อกคงเหลือในระบบ!')
      return
    }
    newQty = oldQty - actionQty.value
  }

  try {
    // 💡 จุดเชื่อม Backend (Supabase):
    /*
    // 1. อัปเดตตาราง MATERIALS
    await supabase.from('MATERIALS').update({ qty_on_hand: newQty }).eq('id', mat.id)

    // 2. บันทึกลงตาราง STOCK_MOVEMENTS
    await supabase.from('STOCK_MOVEMENTS').insert({
      item_type: 'material',
      movement_type: actionType.value === 'in' ? 'receipt' : 'production',
      product_id: mat.id, // สมมติว่าใช้ช่องนี้เก็บไอดีของทั้งสินค้าและวัตถุดิบ
      qty_in: actionType.value === 'in' ? actionQty.value : 0,
      qty_out: actionType.value === 'out' ? actionQty.value : 0,
      balance_after: newQty,
      created_by: 1, // Admin ID
      note: actionNote.value
    })
    */

    // อัปเดต UI จำลอง
    mat.qty_on_hand = newQty
    
    alert(`ทำรายการ ${actionType.value === 'in' ? 'รับเข้า' : 'เบิกออก'} จำนวน ${actionQty.value} ${mat.unit} เรียบร้อยแล้ว`)
    closeModal()

  } catch (error) {
    console.error('เกิดข้อผิดพลาด', error)
  }
}
</script>

<template>
  <div class="p-4 sm:p-6 lg:p-8 bg-gray-50 min-h-screen relative">
    
    <div class="flex flex-col md:flex-row md:items-center justify-between gap-4 mb-8">
      <div>
        <h1 class="text-2xl sm:text-3xl font-bold text-gray-800">Materials Management</h1>
        <p class="text-gray-500 mt-1">ระบบจัดการและเบิกใช้วัตถุดิบ (Raw Materials)</p>
      </div>
      <NuxtLink to="/admin/materials/create" class="bg-blue-600 hover:bg-blue-500 text-white font-bold py-2.5 px-5 rounded-xl shadow-sm transition">
        + เพิ่มวัตถุดิบใหม่
      </NuxtLink>
    </div>

    <div class="bg-white p-4 rounded-2xl shadow-sm border border-gray-100 flex flex-col sm:flex-row gap-4 mb-6">
      <div class="relative flex-1">
        <input 
          v-model="searchQuery"
          type="text" 
          placeholder="ค้นหาชื่อวัตถุดิบ หรือ รหัส..." 
          class="w-full pl-10 pr-4 py-2.5 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-blue-400 outline-none transition"
        />
        <svg class="w-5 h-5 text-gray-400 absolute left-3 top-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
      </div>

      <select v-model="selectedCategory" class="w-full sm:w-48 bg-gray-50 border border-gray-200 text-gray-700 py-2.5 px-4 rounded-xl focus:ring-2 focus:ring-blue-400 outline-none transition cursor-pointer">
        <option value="all">ทุกหมวดหมู่</option>
        <option value="Fabric">ผ้า (Fabric)</option>
        <option value="Filling">ใยยัด (Filling)</option>
        <option value="Hardware">อะไหล่ (Hardware)</option>
        <option value="Thread">ด้าย (Thread)</option>
        <option value="Accessories">ตกแต่ง (Accessories)</option>
      </select>
    </div>

    <div class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse min-w-[900px]">
          <thead>
            <tr class="bg-gray-50 text-gray-600 text-sm border-b border-gray-100">
              <th class="py-4 px-6 font-medium">วัตถุดิบ</th>
              <th class="py-4 px-6 font-medium text-center">คงเหลือ (Qty)</th>
              <th class="py-4 px-6 font-medium text-center">หน่วย (Unit)</th>
              <th class="py-4 px-6 font-medium text-center">สถานะ</th>
              <th class="py-4 px-6 font-medium text-center">รับเข้า / เบิกออก</th>
              <th class="py-4 px-6 font-medium text-right">จัดการ</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-100 text-sm">
            
            <tr v-if="filteredMaterials.length === 0">
              <td colspan="6" class="py-10 text-center text-gray-400">ไม่พบวัตถุดิบ</td>
            </tr>

            <tr v-for="mat in filteredMaterials" :key="mat.id" class="hover:bg-gray-50/50 transition">
              
              <td class="py-3 px-6">
                <div>
                  <p class="font-bold text-gray-800">{{ mat.name }}</p>
                  <p class="text-xs text-gray-400">{{ mat.id }} | ต้นทุน: ฿{{ mat.cost }}/{{ mat.unit }}</p>
                </div>
              </td>
              
              <td class="py-3 px-6 text-center">
                <span class="text-lg font-bold" :class="{'text-red-500': mat.qty_on_hand === 0, 'text-gray-800': mat.qty_on_hand > 0}">
                  {{ mat.qty_on_hand }}
                </span>
              </td>

              <td class="py-3 px-6 text-center text-gray-500 font-medium">{{ mat.unit }}</td>

              <td class="py-3 px-6 text-center">
                <span v-if="mat.qty_on_hand > mat.reorder_level" class="bg-green-100 text-green-700 py-1 px-3 rounded-full text-xs font-bold">มีเพียงพอ</span>
                <span v-else-if="mat.qty_on_hand > 0" class="bg-orange-100 text-orange-700 py-1 px-3 rounded-full text-xs font-bold">ใกล้หมด (ต่ำกว่า {{mat.reorder_level}})</span>
                <span v-else class="bg-red-100 text-red-700 py-1 px-3 rounded-full text-xs font-bold">ของหมด!</span>
              </td>

              <td class="py-3 px-6 text-center">
                <div class="flex items-center justify-center gap-2">
                  <button @click="openActionModal(mat, 'in')" class="bg-green-100 hover:bg-green-200 text-green-700 font-bold py-1 px-3 rounded-lg text-xs transition">
                    + รับเข้า
                  </button>
                  <button @click="openActionModal(mat, 'out')" class="bg-orange-100 hover:bg-orange-200 text-orange-700 font-bold py-1 px-3 rounded-lg text-xs transition">
                    - เบิกใช้
                  </button>
                </div>
              </td>

              <td class="py-3 px-6 text-right whitespace-nowrap space-x-2">
                <button class="p-2 text-gray-400 hover:text-blue-500 bg-white border border-gray-200 rounded-lg shadow-sm transition" title="แก้ไข">
                  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" /></svg>
                </button>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

    <div v-if="isModalOpen" class="fixed inset-0 z-50 flex items-center justify-center p-4 bg-black bg-opacity-50 backdrop-blur-sm transition-opacity">
      <div class="bg-white rounded-2xl shadow-xl w-full max-w-md overflow-hidden transform transition-all">
        
        <div class="px-6 py-4 border-b border-gray-100 flex justify-between items-center" :class="actionType === 'in' ? 'bg-green-50' : 'bg-orange-50'">
          <h3 class="text-lg font-bold" :class="actionType === 'in' ? 'text-green-800' : 'text-orange-800'">
            {{ actionType === 'in' ? 'รับเข้าวัตถุดิบ (Receive)' : 'เบิกใช้วัตถุดิบ (Withdraw)' }}
          </h3>
          <button @click="closeModal" class="text-gray-400 hover:text-gray-600">
            <svg class="w-6 h-6" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M6 18L18 6M6 6l12 12" /></svg>
          </button>
        </div>

        <div class="p-6">
          <p class="text-sm text-gray-500 mb-2">รหัส: {{ selectedMat.id }}</p>
          <p class="font-bold text-gray-800 text-lg mb-6">{{ selectedMat.name }}</p>

          <div class="flex gap-4 mb-6">
            <div class="flex-1 bg-gray-50 p-3 rounded-xl border border-gray-100 text-center">
              <p class="text-xs text-gray-500 mb-1">สต๊อกเดิม</p>
              <p class="text-xl font-bold text-gray-800">{{ selectedMat.qty_on_hand }} <span class="text-sm font-normal">{{ selectedMat.unit }}</span></p>
            </div>
            <div class="flex-1 p-3 rounded-xl border-2 text-center" :class="actionType === 'in' ? 'border-green-200 bg-green-50' : 'border-orange-200 bg-orange-50'">
              <p class="text-xs font-bold mb-1" :class="actionType === 'in' ? 'text-green-600' : 'text-orange-600'">
                {{ actionType === 'in' ? 'จำนวนรับเข้า (+)' : 'จำนวนเบิกออก (-)' }}
              </p>
              <div class="flex items-center justify-center gap-1">
                <input 
                  v-model.number="actionQty" 
                  type="number" 
                  min="0"
                  class="w-20 text-center text-xl font-bold bg-white border border-gray-200 rounded-lg py-1 outline-none"
                  :class="actionType === 'in' ? 'focus:border-green-400 focus:ring-1 focus:ring-green-400' : 'focus:border-orange-400 focus:ring-1 focus:ring-orange-400'"
                />
                <span class="text-sm text-gray-600 font-medium">{{ selectedMat.unit }}</span>
              </div>
            </div>
          </div>

          <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">หมายเหตุ / อ้างอิงการใช้งาน</label>
            <textarea 
              v-model="actionNote" 
              rows="2" 
              class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-2 focus:ring-2 focus:outline-none transition-all resize-none"
              :class="actionType === 'in' ? 'focus:ring-green-400' : 'focus:ring-orange-400'"
              placeholder="เช่น เบิกไปทำตุ๊กตากระต่าย 20 ตัว..."
            ></textarea>
          </div>
          
          <div class="flex gap-3">
            <button @click="closeModal" class="flex-1 py-3 bg-gray-100 text-gray-600 font-bold rounded-xl hover:bg-gray-200 transition">
              ยกเลิก
            </button>
            <button @click="confirmAction" class="flex-1 py-3 text-white font-bold rounded-xl shadow-sm transition" :class="actionType === 'in' ? 'bg-green-600 hover:bg-green-500' : 'bg-orange-600 hover:bg-orange-500'">
              ยืนยันการทำรายการ
            </button>
          </div>

        </div>
      </div>
    </div>

  </div>
</template>