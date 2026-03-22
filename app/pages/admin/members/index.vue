<script setup>
import { ref, onMounted, computed } from 'vue'

definePageMeta({ layout: 'admin' })

// ==========================================
// [BACKEND: เตรียมเชื่อมต่อฐานข้อมูล]
// ==========================================
// const supabase = useSupabaseClient()

// ==========================================
// [STATE: ตัวแปรเก็บข้อมูลและควบคุมหน้าจอ]
// ==========================================
const users = ref([])
const isLoading = ref(true)

// ตัวแปรสำหรับค้นหาและกรองข้อมูล
const searchQuery = ref('')
const filterRole = ref('all') // all, admin, staff, customer
const filterStatus = ref('all') // all, active, inactive

// ==========================================
// [MOCK DATA: จำลองการดึงข้อมูลจากตาราง users]
// ==========================================
const fetchUsers = async () => {
  isLoading.value = true
  
  // 💡 [MOCK DATA] ไม่มี points แล้ว มีแต่ role
  setTimeout(() => {
    users.value = [
      { id: 'u1', username: 'admin_somying', full_name: 'สมหญิง รักงานคราฟต์', email: 'somying@email.com', role: 'admin', status: 'active', created_at: '2025-01-15T08:00:00Z' },
      { id: 'u2', username: 'staff_wipada', full_name: 'วิภาดา ใจสว่าง', email: 'wipada@email.com', role: 'staff', status: 'active', created_at: '2025-02-20T10:30:00Z' },
      { id: 'u3', username: 'mana123', full_name: 'มานะ อดทน', email: 'mana@email.com', role: 'customer', status: 'active', created_at: '2025-03-01T14:15:00Z' },
      { id: 'u4', username: 'piti_user', full_name: 'ปิติ เรียนดี', email: 'piti@email.com', role: 'customer', status: 'inactive', created_at: '2024-11-11T09:00:00Z' }
    ]
    isLoading.value = false
  }, 600)

  /* 
  // 💡 [BACKEND: โค้ดสำหรับต่อ Supabase ของจริง]
  try {
    const { data, error } = await supabase
      .from('users')
      .select('id, username, email, full_name, phone, role, status, created_at')
      .order('created_at', { ascending: false })
    
    if (error) throw error
    users.value = data || []
  } catch (err) {
    console.error('Error fetching users:', err)
  } finally {
    isLoading.value = false
  }
  */
}

// ==========================================
// [LOGIC: กรองข้อมูลแบบ Real-time]
// ==========================================
const filteredUsers = computed(() => {
  return users.value.filter(u => {
    const query = searchQuery.value.toLowerCase()
    const matchSearch = 
      (u.full_name && u.full_name.toLowerCase().includes(query)) || 
      (u.username && u.username.toLowerCase().includes(query)) ||
      (u.email && u.email.toLowerCase().includes(query))

    const matchRole = filterRole.value === 'all' || u.role === filterRole.value
    const matchStatus = filterStatus.value === 'all' || u.status === filterStatus.value

    return matchSearch && matchRole && matchStatus
  })
})

// ==========================================
// [BACKEND LOGIC: ฟังก์ชันแก้ไขสิทธิ์ผู้ใช้งาน]
// ==========================================
const updateRole = async (user, event) => {
  const newRole = event.target.value
  
  // แจ้งเตือนยืนยันการเปลี่ยนสิทธิ์
  if (!confirm(`คุณต้องการเปลี่ยนสิทธิ์ของ "${user.full_name}" เป็น "${newRole.toUpperCase()}" ใช่หรือไม่?`)) {
    // ถ้ากดยกเลิก ให้คืนค่า Select dropdown กลับเป็นค่าเดิม
    event.target.value = user.role 
    return
  }

  try {
    // 💡 [BACKEND] โค้ดอัปเดตลง Supabase ของจริง
    // const { error } = await supabase.from('users').update({ role: newRole }).eq('id', user.id)
    // if (error) throw error

    // อัปเดต UI 
    user.role = newRole
    alert('อัปเดตสิทธิ์ผู้ใช้งานสำเร็จ')
  } catch (err) {
    console.error('Error updating role:', err)
    alert('เกิดข้อผิดพลาดในการเปลี่ยนสิทธิ์')
    event.target.value = user.role // คืนค่ากลับถ้าบันทึกไม่สำเร็จ
  }
}

// สลับสถานะ Active / Inactive
const toggleStatus = async (user) => {
  const newStatus = user.status === 'active' ? 'inactive' : 'active'
  if(!confirm(`ต้องการระงับ/เปิดใช้งานบัญชีของ ${user.full_name} หรือไม่?`)) return
  
  // 💡 [BACKEND] อัปเดตลง Supabase
  // await supabase.from('users').update({ status: newStatus }).eq('id', user.id)
  
  user.status = newStatus
  alert('อัปเดตสถานะสำเร็จ')
}

onMounted(() => {
  fetchUsers()
})
</script>

<template>
  <div class="p-6 max-w-7xl mx-auto">
    
    <!-- [UI: Header & ปุ่มเพิ่มผู้ใช้งาน] -->
    <div class="flex flex-col lg:flex-row lg:items-center justify-between gap-4 mb-8">
      <div>
        <h1 class="text-2xl sm:text-3xl font-bold text-gray-800">ผู้ใช้งาน & สิทธิ์ (Users & Roles)</h1>
        <p class="text-gray-500 mt-1">จัดการบัญชีผู้ใช้งาน กำหนดสิทธิ์แอดมิน พนักงาน และลูกค้า</p>
      </div>
      <button class="bg-[#ff8fa3] hover:bg-pink-400 text-white font-bold py-2.5 px-5 rounded-xl shadow-sm transition whitespace-nowrap">
        + เพิ่มผู้ใช้งานใหม่
      </button>
    </div>

    <!-- [UI: ส่วนค้นหาและตัวกรอง] -->
    <div class="bg-white p-4 rounded-xl shadow-sm border border-gray-100 mb-6 flex flex-col md:flex-row gap-4">
      <div class="flex-1 relative">
        <input 
          v-model="searchQuery" 
          type="text" 
          placeholder="ค้นหาชื่อ, Username, อีเมล..." 
          class="w-full pl-10 pr-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none"
        />
        <svg class="w-5 h-5 text-gray-400 absolute left-3 top-2.5" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z" /></svg>
      </div>
      
      <!-- เปลี่ยนตัวกรองระดับสมาชิก เป็นตัวกรอง "สิทธิ์ (Role)" -->
      <div class="w-full md:w-48">
        <select v-model="filterRole" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-white">
          <option value="all">ทุกสิทธิ์การใช้งาน</option>
          <option value="admin">แอดมิน (Admin)</option>
          <option value="staff">พนักงาน (Staff)</option>
          <option value="customer">ลูกค้า (Customer)</option>
        </select>
      </div>

      <div class="w-full md:w-48">
        <select v-model="filterStatus" class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-[#ff8fa3] outline-none bg-white">
          <option value="all">ทุกสถานะ</option>
          <option value="active">ใช้งานปกติ (Active)</option>
          <option value="inactive">ระงับบัญชี (Inactive)</option>
        </select>
      </div>
    </div>

    <!-- [UI: ตารางแสดงข้อมูลผู้ใช้งาน] -->
    <div class="bg-white rounded-xl shadow-sm border border-gray-100 overflow-hidden">
      <div v-if="isLoading" class="p-10 text-center text-gray-500">กำลังโหลดข้อมูลผู้ใช้งาน...</div>
      
      <div v-else class="overflow-x-auto">
        <table class="w-full text-left border-collapse min-w-[800px]">
          <thead>
            <tr class="bg-gray-50 text-gray-500 text-sm border-b border-gray-100">
              <th class="p-4 font-medium">ข้อมูลผู้ใช้งาน</th>
              <th class="p-4 font-medium">Username</th>
              <th class="p-4 font-medium text-center">สิทธิ์ (Role)</th>
              <th class="p-4 font-medium text-center">สถานะ</th>
              <th class="p-4 font-medium text-center">จัดการ</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="user in filteredUsers" :key="user.id" class="border-b border-gray-50 hover:bg-gray-50 transition">
              
              <td class="p-4">
                <p class="font-bold text-gray-800">{{ user.full_name || 'ไม่ระบุชื่อ' }}</p>
                <p class="text-xs text-gray-500 mt-1">📧 {{ user.email }}</p>
              </td>

              <td class="p-4 text-gray-600 font-medium">
                @{{ user.username || '-' }}
              </td>

              <!-- [UI & LOGIC: Dropdown เพื่อเปลี่ยน Role โดยตรงจากตาราง] -->
              <td class="p-4 text-center">
                <select 
                  :value="user.role" 
                  @change="updateRole(user, $event)"
                  class="px-3 py-1.5 border border-gray-300 rounded-lg text-sm font-bold focus:ring-2 focus:ring-[#ff8fa3] outline-none cursor-pointer"
                  :class="{
                    'bg-purple-50 text-purple-700 border-purple-200': user.role === 'admin',
                    'bg-blue-50 text-blue-700 border-blue-200': user.role === 'staff',
                    'bg-gray-50 text-gray-600': user.role === 'customer'
                  }"
                >
                  <option value="admin">Admin</option>
                  <option value="staff">Staff</option>
                  <option value="customer">Customer</option>
                </select>
              </td>

              <td class="p-4 text-center">
                <span v-if="user.status === 'active'" class="bg-green-100 text-green-700 px-3 py-1 rounded-full text-xs font-bold">Active</span>
                <span v-else class="bg-red-50 text-red-500 border border-red-100 px-3 py-1 rounded-full text-xs font-bold">Inactive</span>
              </td>

              <!-- ปุ่ม Action -->
              <td class="p-4 text-center whitespace-nowrap">
                <button @click="toggleStatus(user)" class="p-2 text-gray-500 hover:bg-gray-100 rounded-lg transition" title="ระงับ/เปิดใช้งาน">
                  <svg v-if="user.status === 'active'" class="w-5 h-5 text-red-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M18.364 18.364A9 9 0 005.636 5.636m12.728 12.728A9 9 0 015.636 5.636m12.728 12.728L5.636 5.636" /></svg>
                  <svg v-else class="w-5 h-5 text-green-500" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" /></svg>
                </button>
              </td>

            </tr>
            <tr v-if="filteredUsers.length === 0">
              <td colspan="5" class="p-10 text-center text-gray-500">ไม่พบรายชื่อผู้ใช้งาน</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>

  </div>
</template>