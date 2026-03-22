<script setup>
import { ref, onMounted, computed } from 'vue'

definePageMeta({ layout: 'admin' })

// ==========================================
// [BACKEND: การเตรียมตัวเชื่อมต่อฐานข้อมูล]
// ==========================================
const supabase = useSupabaseClient()

// [STATE: ตัวแปรสำหรับเก็บข้อมูลและควบคุมหน้าจอ]
const orders = ref([]) // เก็บข้อมูลออเดอร์ที่ดึงมา
const isLoading = ref(true) // ควบคุมสถานะกำลังโหลด
const searchQuery = ref('') // คำค้นหา
const currentTab = ref('all') // Tab สถานะปัจจุบัน

// ==========================================
// [BACKEND: ฟังก์ชันดึงข้อมูล Order จาก Supabase]
// ==========================================
// [MOCK DATA: ฟังก์ชันดึงข้อมูลจำลอง]
// ==========================================
const fetchOrders = async () => {
  isLoading.value = true
  
  // จำลองดีเลย์การโหลดผ่าน Network 0.8 วินาที
  setTimeout(() => {
    // 💡 หากต้องการใช้ข้อมูลจริง ให้ลบ Mock Data ออก แล้วเปิด Comment โค้ด Supabase ด้านล่างแทน
    orders.value = [
      { id: '1111-aaaa', order_no: 'ORD-2603-001', status: 'pending_payment', total_amount: 1500, created_at: '2026-03-19T10:00:00Z', order_source: 'web', members: { full_name: 'สมชาย ใจดี' } },
      { id: '2222-bbbb', order_no: 'ORD-2603-002', status: 'pending_review', total_amount: 850, created_at: '2026-03-18T14:30:00Z', order_source: 'web', members: { full_name: 'วิภาดา รักงานคราฟต์' } },
      { id: '3333-cccc', order_no: 'ORD-2603-003', status: 'paid', total_amount: 2100, created_at: '2026-03-17T09:15:00Z', order_source: 'walk_in', members: null }, // Walk-in จะไม่มี members
      { id: '4444-dddd', order_no: 'ORD-2603-004', status: 'shipped', total_amount: 450, created_at: '2026-03-16T16:45:00Z', order_source: 'web', members: { full_name: 'มานะ อดทน' } },
      { id: '5555-eeee', order_no: 'ORD-2603-005', status: 'cancelled', total_amount: 1200, created_at: '2026-03-15T11:20:00Z', order_source: 'web', members: { full_name: 'ปิติ เรียนดี' } }
    ]
    isLoading.value = false
  }, 800)

  /* โค้ดดึงข้อมูลจริง (ปิดไว้ก่อน)
  try {
    const { data, error } = await supabase
      .from('orders')
      .select('id, order_no, status, total_amount, created_at, order_source, members (full_name)')
      .order('created_at', { ascending: false })
    if (error) throw error
    orders.value = data || []
  } catch (err) {
    console.error(err)
  } finally {
    isLoading.value = false
  }
  */
}

// ==========================================
// [LOGIC: ฟังก์ชันกรองข้อมูลและจัดการ UI]
// ==========================================
// ฟังก์ชันกรองข้อมูลตาม Tab และการค้นหา
const filteredOrders = computed(() => {
  return orders.value.filter(order => {
    // กรองสถานะ
    const matchStatus = currentTab.value === 'all' || order.status === currentTab.value
    // กรองคำค้นหา (ค้นจากเลขที่ออเดอร์ หรือ ชื่อลูกค้า)
    const searchLower = searchQuery.value.toLowerCase()
    const matchSearch = order.order_no.toLowerCase().includes(searchLower) || 
                        (order.members?.full_name || 'Walk-in').toLowerCase().includes(searchLower)
    return matchStatus && matchSearch
  })
})

// กำหนดสีและข้อความของ Badge ตามสถานะ
const statusColors = {
  pending_payment: 'bg-yellow-100 text-yellow-800',
  pending_review: 'bg-orange-100 text-orange-800',
  paid: 'bg-green-100 text-green-800',
  shipped: 'bg-blue-100 text-blue-800',
  cancelled: 'bg-red-100 text-red-800'
}

const statusLabels = {
  pending_payment: 'รอชำระเงิน',
  pending_review: 'รอตรวจสอบสลิป',
  paid: 'ชำระเงินแล้ว',
  shipped: 'จัดส่งแล้ว',
  cancelled: 'ยกเลิก'
}

onMounted(() => {
  fetchOrders() // เรียกดึงข้อมูลเมื่อเปิดหน้าต่าง
})
</script>

<template>
  <div class="p-6 max-w-7xl mx-auto">
    <!-- ========================================== -->
    <!-- [UI: ส่วนหัวและช่องค้นหา]                  -->
    <!-- ========================================== -->
    <div class="flex flex-col sm:flex-row justify-between items-start sm:items-center gap-4 mb-6">
      <h1 class="text-2xl font-bold text-gray-800">จัดการคำสั่งซื้อ</h1>
      <div class="w-full sm:w-72">
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="ค้นหาเลขออเดอร์, ชื่อลูกค้า..." 
          class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none"
        />
      </div>
    </div>

    <!-- ========================================== -->
    <!-- [UI: แท็บสถานะ (Tabs)]                     -->
    <!-- ========================================== -->
    <div class="flex gap-2 overflow-x-auto mb-6 pb-2">
      <button 
        v-for="tab in ['all', 'pending_payment', 'pending_review', 'paid', 'shipped']" 
        :key="tab"
        @click="currentTab = tab"
        class="px-4 py-2 rounded-full text-sm font-medium whitespace-nowrap transition"
        :class="currentTab === tab ? 'bg-[#ff8fa3] text-white' : 'bg-white text-gray-600 border border-gray-200 hover:bg-gray-50'"
      >
        {{ tab === 'all' ? 'ทั้งหมด' : statusLabels[tab] }}
      </button>
    </div>

    <!-- ========================================== -->
    <!-- [UI: ตารางแสดงข้อมูลรายการคำสั่งซื้อ]          -->
    <!-- ========================================== -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
      <!-- กรณีที่ดึงข้อมูลจาก Backend ยังไม่เสร็จ -->
      <div v-if="isLoading" class="p-10 text-center text-gray-500">กำลังโหลดข้อมูล...</div>
      
      <table v-else class="w-full text-left border-collapse">
        <thead>
          <tr class="bg-gray-50 text-gray-500 text-sm border-b border-gray-100">
            <th class="p-4 font-medium">เลขที่คำสั่งซื้อ</th>
            <th class="p-4 font-medium">วันที่</th>
            <th class="p-4 font-medium">ลูกค้า</th>
            <th class="p-4 font-medium">ยอดรวม</th>
            <th class="p-4 font-medium">สถานะ</th>
            <th class="p-4 font-medium text-center">จัดการ</th>
          </tr>
        </thead>
        <tbody>
          <!-- ใช้ v-for ลูปข้อมูลจาก filteredOrders -->
          <tr v-for="order in filteredOrders" :key="order.id" class="border-b border-gray-50 hover:bg-gray-50 transition">
            <td class="p-4 font-medium text-gray-800">{{ order.order_no }}</td>
            <td class="p-4 text-gray-600">{{ new Date(order.created_at).toLocaleDateString('th-TH') }}</td>
            <td class="p-4 text-gray-600">
              {{ order.members?.full_name || 'ลูกค้าทั่วไป' }}
              <span v-if="order.order_source === 'walk_in'" class="ml-2 text-[10px] bg-gray-200 px-2 py-0.5 rounded text-gray-600">Walk-in</span>
            </td>
            <td class="p-4 text-gray-800 font-bold">฿{{ order.total_amount.toLocaleString() }}</td>
            <td class="p-4">
              <!-- แสดงสี Badge อิงจากตัวแปร Object statusColors -->
              <span :class="['px-3 py-1 rounded-full text-xs font-bold', statusColors[order.status]]">
                {{ statusLabels[order.status] }}
              </span>
            </td>
            <td class="p-4 text-center">
              <!-- [ลิงก์ไปหน้า Detail] ยังคงชี้ไปที่โฟลเดอร์ order/ ตามเดิม -->
              <NuxtLink :to="`/admin/order/${order.id}`" class="text-[#ff8fa3] hover:text-pink-700 font-medium text-sm">
                ดูรายละเอียด
              </NuxtLink>
            </td>
          </tr>
          
          <!-- กรณีที่ไม่มีข้อมูลหลังจากค้นหา หรือฐานข้อมูลว่างเปล่า -->
          <tr v-if="filteredOrders.length === 0">
            <td colspan="6" class="p-10 text-center text-gray-500">ไม่พบคำสั่งซื้อ</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>