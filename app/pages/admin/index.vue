<script setup>
import { ref, computed, onMounted } from 'vue'
import { useRouter } from 'vue-router'

definePageMeta({
  layout: 'admin',
})

const router = useRouter()
const supabase = useSupabaseClient() // เรียกใช้ Supabase Client

// ==========================================
// [BACKEND: 1. ข้อมูลสรุป Dashboard]
// ==========================================
const dashboardStats = ref({
  totalSales: 0,
  newOrders: 0,
  pendingReview: 0,
  lowStockAlerts: 0 
})

// ==========================================
// [BACKEND: 2. ข้อมูลรายการคำสั่งซื้อ (ORDERS)]
// ==========================================
const currentTab = ref('all') 
const searchQuery = ref('')
const allOrders = ref([]) // เปลี่ยนเป็น array ว่างเพื่อรอรับข้อมูล
const isLoading = ref(true)

// ฟังก์ชันดึงข้อมูลจาก Database สดๆ
const fetchDashboardData = async () => {
  isLoading.value = true
  try {
    // 1. ดึงออเดอร์ทั้งหมด
    // อ้างอิงจากตาราง orders ใน Database ที่คุณออกแบบไว้
    const { data: ordersData, error } = await supabase
      .from('orders')
      .select('*, members(full_name)') 
      .order('created_at', { ascending: false })
      .limit(50)

    if (error) throw error

    // ทำการ Map ข้อมูลให้ตรงกับ UI
    allOrders.value = ordersData.map(order => ({
      id: order.order_no,
      db_id: order.id, // เก็บ uuid ไว้ใช้อัปเดตสถานะ
      date: new Date(order.created_at).toLocaleString('th-TH'),
      customer: order.members?.full_name || 'ลูกค้าทั่วไป (Walk-in)',
      amount: order.total_amount,
      status: mapStatusForUI(order.status) // สมมติว่ามีฟังก์ชันแปลงสถานะ
    }))

    // 2. คำนวณ Stats (ตัวอย่าง)
    dashboardStats.value.newOrders = ordersData.filter(o => o.status === 'pending_payment').length
    dashboardStats.value.pendingReview = ordersData.filter(o => o.status === 'pending_review').length
    
  } catch (err) {
    console.error('Error fetching dashboard:', err)
  } finally {
    isLoading.value = false
  }
}

// Map สถานะจาก Database ให้ตรงกับ Tab UI
const mapStatusForUI = (dbStatus) => {
  if (dbStatus === 'pending_payment' || dbStatus === 'pending_review') return 'pending'
  if (dbStatus === 'paid' || dbStatus === 'preparing') return 'paid'
  if (dbStatus === 'shipped' || dbStatus === 'completed') return 'shipped'
  if (dbStatus === 'cancelled') return 'cancelled'
  return dbStatus
}

onMounted(() => {
  // fetchDashboardData() // Uncomment เมื่อพร้อมต่อกับ Supabase
  
  // ชั่วคราว: ใส่ Mock Data กลับเข้าไปก่อนระหว่างที่รอ DB จริง
  allOrders.value = [
    { id: 'ORD-005', date: '2026-03-19 09:15', customer: 'วิภาดา ใจสว่าง', amount: 890, status: 'pending' },
    { id: 'ORD-004', date: '2026-03-18 20:30', customer: 'มานะ รักดี', amount: 450, status: 'pending' },
    { id: 'ORD-003', date: '2026-03-18 14:20', customer: 'สมหญิง รักงานคราฟต์', amount: 357, status: 'paid' },
  ]
  dashboardStats.value = { totalSales: 45200, newOrders: 18, pendingReview: 5, lowStockAlerts: 3 }
  isLoading.value = false
})

const filteredOrders = computed(() => {
  return allOrders.value.filter(order => {
    const matchStatus = currentTab.value === 'all' || order.status === currentTab.value
    const matchSearch = order.id.toLowerCase().includes(searchQuery.value.toLowerCase()) || 
                        order.customer.toLowerCase().includes(searchQuery.value.toLowerCase())
    return matchStatus && matchSearch
  })
})

// ==========================================
// [เชื่อมต่อกับหน้า Order Details]
// ==========================================
const viewOrderDetails = (orderId) => {
  router.push(`/admin/orders/${orderId}`)
}

const quickApprove = async (id) => {
  if (confirm(`ยืนยันการอนุมัติออเดอร์ ${id} อย่างรวดเร็วหรือไม่?`)) {
    // โค้ดสำหรับอัปเดตสถานะใน DB (เตรียมไว้)
    /*
    const order = allOrders.value.find(o => o.id === id)
    await supabase.from('orders').update({ status: 'paid' }).eq('id', order.db_id)
    */
    
    // อัปเดต UI ทันที
    const orderIndex = allOrders.value.findIndex(o => o.id === id)
    if (orderIndex !== -1) allOrders.value[orderIndex].status = 'paid'
  }
}
</script>







<template>
  <div class="p-4 sm:p-6 lg:p-8 bg-gray-50 min-h-screen">
    
  
    <!-- ส่วนหัวเรื่อง Dashboard -->
    <div class="mb-8">
      <h1 class="text-3xl font-bold text-gray-800">Dashboard Overview</h1>
      <p class="text-gray-500 mt-1">สรุปยอดขายและจัดการคำสั่งซื้อล่าสุด</p>
    </div>

    <!-- 4 ช่องสรุปข้อมูล (แสดงผลเท่านั้น ไม่สามารถคลิกได้) -->
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
      
      <!-- ช่อง 1: ยอดขายรวม -->
      <div class="bg-white p-6 rounded-2xl border border-gray-100 shadow-sm flex items-center gap-5">
        <div class="w-14 h-14 rounded-full bg-green-100 flex items-center justify-center flex-shrink-0">
          <svg class="w-7 h-7 text-green-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 8c-1.657 0-3 .895-3 2s1.343 2 3 2 3 .895 3 2-1.343 2-3 2m0-8c1.11 0 2.08.402 2.599 1M12 8V7m0 1v8m0 0v1m0-1c-1.11 0-2.08-.402-2.599-1M21 12a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
        </div>
        <div>
          <p class="text-sm font-bold text-gray-500">ยอดขายรวม <br><span class="font-normal">(เดือนนี้)</span></p>
          <p class="text-2xl font-black text-gray-800 mt-1">฿{{ dashboardStats.totalSales.toLocaleString() }}</p>
        </div>
      </div>

      <!-- ช่อง 2: ออเดอร์ใหม่ -->
      <div class="bg-white p-6 rounded-2xl border border-blue-100 shadow-sm flex items-center gap-5">
        <div class="w-14 h-14 rounded-full bg-blue-100 flex items-center justify-center flex-shrink-0">
          <svg class="w-7 h-7 text-blue-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M16 11V7a4 4 0 00-8 0v4M5 9h14l1 12H4L5 9z" />
          </svg>
        </div>
        <div>
          <p class="text-sm font-bold text-gray-500">ออเดอร์ใหม่</p>
          <p class="text-2xl font-black text-gray-800 mt-1">{{ dashboardStats.newOrders }}</p>
        </div>
      </div>

      <!-- ช่อง 3: รอตรวจสอบสลิป (ดีไซน์สีเหลืองเด่น) -->
      <div class="bg-white p-6 rounded-2xl border border-yellow-300 shadow-sm flex items-center gap-5">
        <div class="w-14 h-14 rounded-full bg-yellow-200 flex items-center justify-center flex-shrink-0">
          <svg class="w-7 h-7 text-yellow-600" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
          </svg>
        </div>
        <div>
          <p class="text-sm font-bold text-yellow-600">รอตรวจสอบสลิป!</p>
          <p class="text-2xl font-black text-gray-800 mt-1">
            {{ dashboardStats.pendingReview }} <span class="text-base font-bold text-gray-600">รายการ</span>
          </p>
        </div>
      </div>

      <!-- ช่อง 4: สต๊อกใกล้หมด (ดีไซน์เตือนสีแดง) -->
      <div class="bg-white p-6 rounded-2xl border border-red-300 shadow-sm flex items-center gap-5">
        <div class="w-14 h-14 rounded-full bg-red-200 flex items-center justify-center flex-shrink-0">
          <svg class="w-7 h-7 text-red-500" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 9v2m0 4h.01m-6.938 4h13.856c1.54 0 2.502-1.667 1.732-3L13.732 4c-.77-1.333-2.694-1.333-3.464 0L3.34 16c-.77 1.333.192 3 1.732 3z" />
          </svg>
        </div>
        <div>
          <p class="text-sm font-bold text-red-500">สต๊อกใกล้หมด</p>
          <p class="text-2xl font-black text-gray-800 mt-1">
            {{ dashboardStats.lowStockAlerts }} <span class="text-base font-bold text-gray-600">รายการ</span>
          </p>
        </div>
      </div>

    </div>

    <!-- พื้นที่ด้านล่างสำหรับใส่ตารางออเดอร์ล่าสุด หรือกราฟ -->
    <!-- <div class="bg-white rounded-xl shadow-sm border border-gray-100...">...</div> -->

  


    <!-- ตารางออเดอร์ -->
    <div class="bg-white rounded-2xl shadow-sm border border-gray-100 overflow-hidden">
      
      <div class="p-6 border-b border-gray-100 flex flex-col md:flex-row md:items-center justify-between gap-4">
        <h2 class="text-lg font-bold text-gray-800">จัดการคำสั่งซื้อ (Orders)</h2>
        
        <div class="relative w-full md:w-64">
          <input 
            v-model="searchQuery"
            type="text" 
            placeholder="ค้นหารหัสออเดอร์ หรือ ชื่อลูกค้า..." 
            class="w-full pl-10 pr-4 py-2 border border-gray-200 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none"
          />
          <svg class="w-5 h-5 text-gray-400 absolute left-3 top-2.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
        </div>
      </div>

      <div class="px-6 pt-4 border-b border-gray-100 flex gap-6 overflow-x-auto custom-scrollbar">
        <button @click="currentTab = 'all'" :class="currentTab === 'all' ? 'border-[#ff8fa3] text-[#ff8fa3] font-bold' : 'border-transparent text-gray-500 hover:text-gray-700'" class="pb-3 border-b-2 whitespace-nowrap transition-colors">ทั้งหมด</button>
        <button @click="currentTab = 'pending'" :class="currentTab === 'pending' ? 'border-yellow-500 text-yellow-600 font-bold' : 'border-transparent text-gray-500 hover:text-gray-700'" class="pb-3 border-b-2 whitespace-nowrap transition-colors flex items-center gap-2">รอตรวจสอบ <span class="bg-yellow-100 text-yellow-600 py-0.5 px-2 rounded-full text-xs">{{ dashboardStats.pendingReview }}</span></button>
        <button @click="currentTab = 'paid'" :class="currentTab === 'paid' ? 'border-blue-500 text-blue-600 font-bold' : 'border-transparent text-gray-500 hover:text-gray-700'" class="pb-3 border-b-2 whitespace-nowrap transition-colors">เตรียมจัดส่ง (Paid)</button>
        <button @click="currentTab = 'shipped'" :class="currentTab === 'shipped' ? 'border-green-500 text-green-600 font-bold' : 'border-transparent text-gray-500 hover:text-gray-700'" class="pb-3 border-b-2 whitespace-nowrap transition-colors">จัดส่งแล้ว</button>
      </div>

      <div class="overflow-x-auto">
        <table class="w-full text-left border-collapse min-w-[800px]">
          <thead>
            <tr class="bg-gray-50 text-gray-600 text-sm border-b border-gray-100">
              <th class="py-4 px-6 font-medium whitespace-nowrap">รหัสออเดอร์</th>
              <th class="py-4 px-6 font-medium whitespace-nowrap">วัน/เวลา</th>
              <th class="py-4 px-6 font-medium whitespace-nowrap">ลูกค้า</th>
              <th class="py-4 px-6 font-medium whitespace-nowrap">ยอดรวม</th>
              <th class="py-4 px-6 font-medium whitespace-nowrap text-center">สถานะ</th>
              <th class="py-4 px-6 font-medium whitespace-nowrap text-right">จัดการ</th>
            </tr>
          </thead>
          <tbody class="divide-y divide-gray-100 text-sm">
            
            <tr v-if="filteredOrders.length === 0">
              <td colspan="6" class="py-8 text-center text-gray-400">ไม่พบคำสั่งซื้อ</td>
            </tr>

            <tr v-for="order in filteredOrders" :key="order.id" class="hover:bg-gray-50/50 transition">
              <td class="py-3 px-6 font-bold text-[#ff8fa3] whitespace-nowrap">{{ order.id }}</td>
              <td class="py-3 px-6 text-gray-500 whitespace-nowrap">{{ order.date }}</td>
              <td class="py-3 px-6 text-gray-700 font-medium whitespace-nowrap">{{ order.customer }}</td>
              <td class="py-3 px-6 font-bold text-gray-800 whitespace-nowrap">฿{{ order.amount.toLocaleString() }}</td>
              
              <td class="py-3 px-6 text-center whitespace-nowrap">
                <span v-if="order.status === 'pending'" class="bg-yellow-100 text-yellow-700 py-1 px-3 rounded-full text-xs font-bold animate-pulse">รอตรวจสอบสลิป</span>
                <span v-else-if="order.status === 'paid'" class="bg-blue-100 text-blue-700 py-1 px-3 rounded-full text-xs font-bold">เตรียมจัดส่ง</span>
                <span v-else-if="order.status === 'shipped'" class="bg-green-100 text-green-700 py-1 px-3 rounded-full text-xs font-bold">จัดส่งแล้ว</span>
                <span v-else-if="order.status === 'cancelled'" class="bg-red-100 text-red-700 py-1 px-3 rounded-full text-xs font-bold">ยกเลิก</span>
              </td>
              
              <td class="py-3 px-6 text-right whitespace-nowrap space-x-2">
                
                <!-- ปุ่มเชื่อมไปหน้า Order Details -->
                <button 
                  @click="viewOrderDetails(order.id)"
                  class="p-2 text-white bg-gray-800 hover:bg-gray-700 rounded-lg shadow-sm transition flex items-center justify-center gap-1 inline-flex" title="ตรวจสอบรายละเอียด"
                >
                  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15 12a3 3 0 11-6 0 3 3 0 016 0z" /><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z" /></svg>
                  <span class="text-xs font-bold pr-1">ตรวจสลิป</span>
                </button>
                
                <button 
                  v-if="order.status === 'pending'" 
                  @click="quickApprove(order.id)"
                  class="p-2 text-green-600 hover:bg-green-50 border border-green-200 rounded-lg shadow-sm transition" title="อนุมัติด่วน"
                >
                  <svg class="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" /></svg>
                </button>

              </td>
            </tr>

          </tbody>
        </table>
      </div>
      
    </div>

  </div>
</template>

<style scoped>
.custom-scrollbar::-webkit-scrollbar {
  display: none;
}
.custom-scrollbar {
  -ms-overflow-style: none;
  scrollbar-width: none;
}
</style>