<script setup>
import { ref, onMounted } from 'vue'

// ==========================================
// [BACKEND: ส่วนที่ 1 - เตรียมตัวแปร]
// ==========================================
// สร้างตัวแปรสำหรับเก็บข้อมูลที่จะแสดงในฟอร์ม
const profileForm = ref({
  firstName: '',
  lastName: '',
  email: '',
  mobile: ''
})

// ตัวแปรสำหรับเปิด/ปิดเมนูบนจอมือถือ
const isMobileMenuOpen = ref(false)

// ==========================================
// [BACKEND: ส่วนที่ 2 - ดึงข้อมูลผู้ใช้ (Fetch Data)]
// ==========================================
// onMounted จะทำงานทันทีเมื่อเปิดหน้านี้ขึ้นมา
onMounted(async () => {
  try {
    // 💡 จุดเชื่อม Backend: ตรงนี้คุณจะใช้ Supabase ดึงข้อมูลลูกค้าที่ Login อยู่
    // ตัวอย่างโค้ดของจริง (ถ้าใช้ @nuxtjs/supabase):
    // const supabase = useSupabaseClient()
    // const user = useSupabaseUser()
    // const { data, error } = await supabase.from('users').select('*').eq('id', user.value.id).single()
    // 
    // if (data) {
    //   profileForm.value = {
    //     firstName: data.first_name,
    //     lastName: data.last_name,
    //     email: data.email,
    //     mobile: data.phone
    //   }
    // }

    // 👇 โค้ดจำลอง (Mock Data) ไปก่อนเพื่อให้เห็นภาพ
    profileForm.value = {
      firstName: 'สมหญิง',
      lastName: 'รักงานคราฟต์',
      email: 'somying@example.com',
      mobile: '0812345678'
    }
  } catch (error) {
    console.error('ไม่สามารถดึงข้อมูลโปรไฟล์ได้:', error)
  }
})

// ==========================================
// [BACKEND: ส่วนที่ 3 - อัปเดตข้อมูล (Update Data)]
// ==========================================
const saveChanges = async () => {
  // เช็กว่ากรอกข้อมูลครบไหม
  if (!profileForm.value.firstName || !profileForm.value.lastName) {
    alert('กรุณากรอกชื่อและนามสกุลให้ครบถ้วน')
    return
  }

  try {
    // 💡 จุดเชื่อม Backend: ส่งข้อมูลที่แก้ไปบันทึกลง Database
    // ตัวอย่างโค้ดของจริง:
    // const supabase = useSupabaseClient()
    // const user = useSupabaseUser()
    // const { error } = await supabase.from('users').update({
    //   first_name: profileForm.value.firstName,
    //   last_name: profileForm.value.lastName,
    //   phone: profileForm.value.mobile
    // }).eq('id', user.value.id)
    // 
    // if (error) throw error

    // 👇 โค้ดจำลองเมื่อบันทึกสำเร็จ
    console.log('อัปเดตข้อมูลส่งไป Backend:', profileForm.value)
    alert('บันทึกข้อมูลโปรไฟล์สำเร็จ!')
  } catch (error) {
    console.error('เกิดข้อผิดพลาดในการบันทึก:', error)
    alert('ไม่สามารถบันทึกข้อมูลได้ กรุณาลองใหม่อีกครั้ง')
  }
}

// ฟังก์ชันปุ่ม Cancel (รีเซ็ตกลับไปเป็นค่าเดิม)
const cancelChanges = () => {
  // 💡 จุดเชื่อม Backend: อาจจะเรียกฟังก์ชันดึงข้อมูลจาก Database อีกรอบเพื่อคืนค่าเดิม
  alert('ยกเลิกการแก้ไข')
}
</script>

<template>
  <div class="min-h-screen bg-[#f8f9fa] py-8 px-4 sm:px-6 lg:px-8">
    <div class="max-w-7xl mx-auto">
      
      <div class="grid grid-cols-1 md:grid-cols-4 gap-8">
        
        <div class="md:col-span-1">
          
          <button 
            @click="isMobileMenuOpen = !isMobileMenuOpen"
            class="md:hidden w-full flex items-center justify-between bg-white px-4 py-3 rounded-xl shadow-sm font-bold text-gray-800 mb-4"
          >
            เมนูบัญชีของฉัน
            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 transform transition-transform" :class="{'rotate-180': isMobileMenuOpen}" viewBox="0 0 20 20" fill="currentColor">
              <path fill-rule="evenodd" d="M5.293 7.293a1 1 0 011.414 0L10 10.586l3.293-3.293a1 1 0 111.414 1.414l-4 4a1 1 0 01-1.414 0l-4-4a1 1 0 010-1.414z" clip-rule="evenodd" />
            </svg>
          </button>

          <div :class="{'hidden md:block': !isMobileMenuOpen, 'block': isMobileMenuOpen}" class="bg-white md:bg-transparent rounded-xl shadow-sm md:shadow-none p-4 md:p-0">
            
            <div class="mb-6">
              <h3 class="font-bold text-gray-800 mb-3 pl-2">Manage My Account</h3>
              <ul class="space-y-2 text-gray-500">
                <li>
                  <NuxtLink 
                    to="/profile" 
                    class="block rounded-lg px-2 py-1.5 transition hover:text-[#ff8fa3]"
                    exact-active-class="text-[#ff8fa3] font-bold bg-pink-50"
                  >
                    My Profile
                  </NuxtLink>
                </li>
                <li>
                  <NuxtLink 
                    to="/profile/address" 
                    class="block rounded-lg px-2 py-1.5 transition hover:text-[#ff8fa3]"
                    active-class="text-[#ff8fa3] font-bold bg-pink-50"
                  >
                    Address Book
                  </NuxtLink>
                </li>
                <li>
                  <NuxtLink 
                    to="/profile/payment" 
                    class="block rounded-lg px-2 py-1.5 transition hover:text-[#ff8fa3]"
                    active-class="text-[#ff8fa3] font-bold bg-pink-50"
                  >
                    My Payment Options
                  </NuxtLink>
                </li>
              </ul>
            </div>

            <div class="mb-6">
              <h3 class="font-bold text-gray-800 mb-3 pl-2">My Orders</h3>
              <ul class="space-y-2 text-gray-500">
                <li>
                  <NuxtLink 
                    to="/orders" 
                    class="block rounded-lg px-2 py-1.5 transition hover:text-[#ff8fa3]"
                    active-class="text-[#ff8fa3] font-bold bg-pink-50"
                  >
                    ประวัติคำสั่งซื้อ
                  </NuxtLink>
                </li>
                <li>
                  <NuxtLink 
                    to="/orders/returns" 
                    class="block rounded-lg px-2 py-1.5 transition hover:text-[#ff8fa3]"
                    active-class="text-[#ff8fa3] font-bold bg-pink-50"
                  >
                    My Returns
                  </NuxtLink>
                </li>
              </ul>
            </div>

            <div class="mb-6">
              <NuxtLink to="/reviews" class="font-bold text-gray-800 hover:text-[#ff8fa3] pl-2 transition">My Reviews</NuxtLink>
            </div>

            <div>
              <NuxtLink to="/wishlist" class="font-bold text-gray-800 hover:text-[#ff8fa3] pl-2 transition">My Wishlist</NuxtLink>
            </div>

          </div>
        </div>

        <div class="md:col-span-3">
          <div class="bg-white rounded-2xl shadow-sm p-6 md:p-10">
            
            <h2 class="text-2xl font-bold text-gray-800 mb-8 border-b border-gray-100 pb-4">Edit Profile</h2>
            
            <form @submit.prevent="saveChanges" class="space-y-6">
              
              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">First Name</label>
                  <input 
                    v-model="profileForm.firstName" 
                    type="text" 
                    class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
                  />
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">Last Name</label>
                  <input 
                    v-model="profileForm.lastName" 
                    type="text" 
                    class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
                  />
                </div>
              </div>

              <div class="grid grid-cols-1 md:grid-cols-2 gap-6">
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">Email</label>
                  <input 
                    v-model="profileForm.email" 
                    type="email" 
                    readonly
                    class="w-full bg-gray-100 border border-gray-200 text-gray-500 rounded-xl px-4 py-3 cursor-not-allowed outline-none"
                  />
                  <p class="text-xs text-gray-400 mt-1">* อีเมลไม่สามารถเปลี่ยนได้</p>
                </div>
                <div>
                  <label class="block text-sm font-medium text-gray-700 mb-2">Mobile</label>
                  <input 
                    v-model="profileForm.mobile" 
                    type="tel" 
                    class="w-full bg-gray-50 border border-gray-200 rounded-xl px-4 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
                  />
                </div>
              </div>

              <div class="flex flex-col sm:flex-row justify-end gap-4 pt-8">
                <button 
                  type="button" 
                  @click="cancelChanges"
                  class="w-full sm:w-auto px-8 py-3 bg-white border-2 border-gray-200 text-gray-600 font-bold rounded-xl hover:bg-gray-50 hover:border-gray-300 transition-all"
                >
                  CANCEL
                </button>
                <button 
                  type="submit" 
                  class="w-full sm:w-auto px-8 py-3 bg-[#ff8fa3] text-white font-bold rounded-xl hover:bg-pink-400 hover:shadow-md transition-all"
                >
                  SAVE CHANGES
                </button>
              </div>

            </form>

          </div>
        </div>

      </div>

    </div>
  </div>
</template>