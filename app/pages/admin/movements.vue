<script setup>
import { ref, computed } from 'vue'

definePageMeta({
  layout: 'admin',
})

const searchQuery = ref('')
const filterItemType = ref('all')     // 'all', 'product', 'material'
const filterMovement = ref('all') // 'all', 'order', 'adjustment', 'production', 'receipt'

// ==========================================
// [BACKEND: ดึงข้อมูลจากตาราง STOCK_MOVEMENTS]
// ==========================================
// จำลองข้อมูลให้ตรงกับ Database Diagram ของคุณเป๊ะๆ
const movementLogs = ref([
  { 
    id: 5, created_at: '2026-03-19 10:15:00', item_type: 'product', movement_type: 'adjustment', 
    item_name: 'สติกเกอร์น้องกระต่ายพาสเทล', item_code: 'PRD-001', 
    qty_in: 5, qty_out: 0, balance_after: 50, 
    created_by_name: 'Admin Somying', note: 'ปรับปรุงสต๊อกจากการตรวจนับ (Manual)', ref_table: null, ref_id: null
  },
  { 
    id: 4, created_at: '2026-03-18 14:30:22', item_type: 'product', movement_type: 'order', 
    item_name: 'พวงกุญแจดอกทิวลิป', item_code: 'PRD-002', 
    qty_in: 0, qty_out: 2, balance_after: 8, 
    created_by_name: 'System', note: 'ตัดสต๊อกอัตโนมัติ', ref_table: 'orders', ref_id: 'ORD-002'
  },
  { 
    id: 3, created_at: '2026-03-17 09:00:11', item_type: 'material', movement_type: 'receipt', 
    item_name: 'ผ้าคอตตอนสีชมพู', item_code: 'MAT-005', 
    qty_in: 100, qty_out: 0, balance_after: 150, 
    created_by_name: 'Admin Mana', note: 'รับเข้าวัตถุดิบจาก Supplier', ref_table: 'material_receipts', ref_id: 'REC-012'
  },
  { 
    id: 2, created_at: '2026-03-16 16:45:00', item_type: 'material', movement_type: 'production', 
    item_name: 'ใยสังเคราะห์', item_code: 'MAT-002', 
    qty_in: 0, qty_out: 5, balance_after: 20, 
    created_by_name: 'Admin Somying', note: 'เบิกไปผลิตยางรัดผม', ref_table: 'production_logs', ref_id: 'PROD-008'
  },
  { 
    id: 1, created_at: '2026-03-15 11:20:05', item_type: 'product', movement_type: 'production', 
    item_name: 'ยางรัดผม + ตุ๊กตาปลา', item_code: 'PRD-004', 
    qty_in: 20, qty_out: 0, balance_after: 120, 
    created_by_name: 'Admin Somying', note: 'รับเข้าจากการผลิตเสร็จสิ้น', ref_table: 'production_logs', ref_id: 'PROD-008'
  },
])

// ฟังก์ชันกรองข้อมูล
const filteredLogs = computed(() => {
  return movementLogs.value.filter(log => {
    // 1. ค้นหาข้อความ
    const matchSearch = log.item_name.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
                        log.item_code.toLowerCase().includes(searchQuery.value.toLowerCase()) ||
                        (log.ref_id && log.ref_id.toLowerCase().includes(searchQuery.value.toLowerCase()))
    
    // 2. กรองประเภท Item (Product / Material)
    const matchItemType = filterItemType.value === 'all' || log.item_type === filterItemType.value

    // 3. กรองประเภทความเคลื่อนไหว (Order, Adjustment, Production...)
    const matchMovement = filterMovement.value === 'all' || log.movement_type === filterMovement.value

    return matchSearch && matchItemType && matchMovement
  })
})

// ฟังก์ชันช่วยจัดรูปแบบวันที่
const formatDate = (dateString) => {
  const date = new Date(dateString)
  return {
    date: date.toLocaleDateString('th-TH', { day: '2-digit', month: 'short', year: 'numeric' }),
    time: date.toLocaleTimeString('th-TH', { hour: '2-digit', minute: '2-digit' })
  }
}
</script>

<template>
  <div class="p-4 sm:p-6 lg:p-8 bg-gray-50 min-h-screen">
    
    <!-- ส่วนหัว -->
    <div class="mb-8">
      <h1 class="text-2xl sm:text-3xl font-bold text-gray-800">Stock Movements</h1>
      <p class="text-gray-500 mt-1">ประวัติการเคลื่อนไหวของสินค้าและวัตถุดิบทั้งหมด</p>
    </div>

    <!-- แถบเครื่องมือค้นหาและกรอง -->
    <div class="bg-white p-4 rounded-2xl shadow-sm border border-gray-100 flex flex-col sm:flex-row gap-4 mb-6">
      
      <div class="relative flex-1">
        <input 
          v-model="searchQuery"
          type="text" 
          placeholder="ค้นหาชื่อ, รหัสรายการ, หรือเลขอ้างอิง..." 
          class="w-full pl-10 pr-4 py-2.5 bg-gray-50 border border-gray-200 rounded-xl focus:ring-2 focus:ring-[#ff8fa3] outline-none transition"
        />
        <svg class="w-5 h-5 text-gray-400 absolute left-3 top-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
      </div>

      <select v-model="filterItemType" class="w-full sm:w-40 bg-gray-50 border border-gray-200 text-gray-700 py-2.5 px-4 rounded-xl focus:ring-2 focus:ring-[#ff8fa3] outline-none transition cursor-pointer">
        <option value="all">ทุกประเภทรายการ</option>
        <option value="product">สินค้า (Products)</option>
        <option value="material">วัตถุดิบ (Materials)</option>
      </select>

      <select v-model="filterMovement" class="w-full sm:w-48 bg-gray-50 border border-gray-200 text-gray-700 py-2.5 px-4 rounded-xl focus:ring-2 focus:ring-[#ff8fa3] outline-none transition cursor-pointer">
        <option value="all">ทุกการเคลื่อนไหว</option>
        <option value="order">ขายออก (Order)</option>
        <option value="receipt">รับของเข้า (Receipt)</option>
        <option value="production">ผลิต/เบิก (Production)</option>
        <option value="adjustment">ปรับปรุงยอด (Adjustment)</option>
      </select>

    </div>

    <!-- ตารางประวัติการเคลื่อนไหว -->
    <div class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse min-w-[1000px]">
          <thead>
            <tr class="bg-gray-50 text-gray-600 text-sm border-b border-gray-100">
              <th class="py-4 px-4 font-medium">วัน/เวลา</th>
              <th class="py-4 px-4 font-medium">รายการ (Item)</th>
              <th class="py-4 px-4 font-medium text-center">เหตุผล (Type)</th>
              <th class="py-4 px-4 font-medium text-center text-green-600">เข้า (In)</th>
              <th class="py-4 px-4 font-medium text-center text-red-500">ออก (Out)</th>
              <th class="py-4 px-4 font-medium text-center">คงเหลือ (Bal)</th>
              <th class="py-4 px-4 font-medium">หมายเหตุ / อ้างอิง</th>
              <th class="py-4 px-4 font-medium">ผู้ทำรายการ</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-100 text-sm">
            
            <tr v-if="filteredLogs.length === 0">
              <td colspan="8" class="py-10 text-center text-gray-400">ไม่พบประวัติการเคลื่อนไหวที่ตรงตามเงื่อนไข</td>
            </tr>

            <tr v-for="log in filteredLogs" :key="log.id" class="hover:bg-gray-50/50 transition">
              
              <td class="py-3 px-4 text-gray-500 whitespace-nowrap">
                <div class="flex flex-col">
                  <span class="font-medium text-gray-800">{{ formatDate(log.created_at).date }}</span>
                  <span class="text-xs">{{ formatDate(log.created_at).time }} น.</span>
                </div>
              </td>
              
              <td class="py-3 px-4">
                <div class="flex items-start gap-2">
                  <!-- โชว์ Badge เล็กๆ ว่าเป็นสินค้าหรือวัตถุดิบ -->
                  <span v-if="log.item_type === 'product'" class="mt-0.5 w-2 h-2 rounded-full bg-pink-400 flex-shrink-0" title="Product"></span>
                  <span v-else class="mt-0.5 w-2 h-2 rounded-full bg-blue-400 flex-shrink-0" title="Material"></span>
                  
                  <div>
                    <p class="font-bold text-gray-800">{{ log.item_name }}</p>
                    <p class="text-xs text-gray-400">{{ log.item_code }}</p>
                  </div>
                </div>
              </td>
              
              <td class="py-3 px-4 text-center">
                <span v-if="log.movement_type === 'order'" class="bg-orange-100 text-orange-700 py-1 px-2.5 rounded-md text-xs font-bold">Order</span>
                <span v-else-if="log.movement_type === 'receipt'" class="bg-green-100 text-green-700 py-1 px-2.5 rounded-md text-xs font-bold">Receipt</span>
                <span v-else-if="log.movement_type === 'production'" class="bg-purple-100 text-purple-700 py-1 px-2.5 rounded-md text-xs font-bold">Production</span>
                <span v-else class="bg-gray-200 text-gray-700 py-1 px-2.5 rounded-md text-xs font-bold">Adjustment</span>
              </td>

              <!-- จำนวนเข้า (In) -->
              <td class="py-3 px-4 text-center">
                <span v-if="log.qty_in > 0" class="text-green-600 font-bold bg-green-50 px-2 py-1 rounded">+{{ log.qty_in }}</span>
                <span v-else class="text-gray-300">-</span>
              </td>

              <!-- จำนวนออก (Out) -->
              <td class="py-3 px-4 text-center">
                <span v-if="log.qty_out > 0" class="text-red-500 font-bold bg-red-50 px-2 py-1 rounded">-{{ log.qty_out }}</span>
                <span v-else class="text-gray-300">-</span>
              </td>

              <!-- ยอดคงเหลือ (Balance After) -->
              <td class="py-3 px-4 text-center text-gray-800 font-bold">{{ log.balance_after }}</td>
              
              <td class="py-3 px-4">
                <div class="flex flex-col">
                  <span v-if="log.ref_id" class="text-xs font-bold text-blue-600 mb-0.5 hover:underline cursor-pointer">{{ log.ref_id }}</span>
                  <span class="text-xs text-gray-500 truncate max-w-[150px] lg:max-w-[200px]" :title="log.note">{{ log.note || '-' }}</span>
                </div>
              </td>

              <td class="py-3 px-4">
                <div class="flex items-center gap-2">
                  <div class="w-6 h-6 rounded-full bg-gray-200 flex items-center justify-center text-xs text-gray-500 flex-shrink-0">
                    <svg class="w-3 h-3" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 7a4 4 0 11-8 0 4 4 0 018 0zM12 14a7 7 0 00-7 7h14a7 7 0 00-7-7z" /></svg>
                  </div>
                  <span class="text-gray-700 font-medium text-xs">{{ log.created_by_name }}</span>
                </div>
              </td>

            </tr>
          </tbody>
        </table>
      </div>
      
      <!-- Pagination -->
      <div class="p-4 border-t border-gray-100 flex items-center justify-between text-sm text-gray-500">
        <span>แสดง 1 ถึง {{ filteredLogs.length }} จาก {{ filteredLogs.length }} รายการ</span>
        <div class="flex gap-2">
          <button class="px-3 py-1 border border-gray-200 rounded-lg hover:bg-gray-50 disabled:opacity-50" disabled>ก่อนหน้า</button>
          <button class="px-3 py-1 border border-gray-200 rounded-lg bg-[#ff8fa3] text-white">1</button>
          <button class="px-3 py-1 border border-gray-200 rounded-lg hover:bg-gray-50 disabled:opacity-50" disabled>ถัดไป</button>
        </div>
      </div>

    </div>

  </div>
</template>