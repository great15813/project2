<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'

const router = useRouter()

// ตัวแปรสำหรับสลับโหมด (true = ล็อกอิน, false = สมัครสมาชิก)
const isLogin = ref(true)

// [BACKEND: ส่วนที่ 1 - ตัวแปรเก็บข้อมูลฟอร์ม]
const form = ref({
  username: '', // ใช้เฉพาะตอนสมัครสมาชิก
  email: '',
  password: '',
  confirmPassword: '' // ใช้เฉพาะตอนสมัครสมาชิก
})

// [BACKEND: ส่วนที่ 2 - ฟังก์ชันเมื่อกดปุ่ม Submit (Sign In / Sign Up)]
const handleSubmit = async () => {
  if (isLogin.value) {
    // ==========================================
    // 🟢 โหมด LOGIN
    // ==========================================
    if (!form.value.email || !form.value.password) {
      alert('กรุณากรอกอีเมลและรหัสผ่านให้ครบถ้วน')
      return
    }

    try {
      // 💡 จุดเชื่อม Backend (Supabase):
      // const supabase = useSupabaseClient()
      // const { data, error } = await supabase.auth.signInWithPassword({
      //   email: form.value.email,
      //   password: form.value.password
      // })
      // if (error) throw error

      // สมมติว่าดึง Role จาก Database มาได้เป็น 'admin' หรือ 'customer'
      // ลองเปลี่ยนค่าตัวแปรนี้เป็น 'customer' ดูครับ แล้วระบบจะพาไปอีกหน้า
      const userRole = 'admin'


      console.log('ล็อกอินสำเร็จด้วย:', form.value.email)
      alert('เข้าสู่ระบบสำเร็จ!')
    //ระบบนำทาง (Routing) ตามสิทธิ์การใช้งาน
      if (userRole === 'admin') {
        router.push('/admin') // ถ้าเป็นแอดมิน พาเข้าหลังบ้าน
      } else {
        router.push('/') // ถ้าเป็นลูกค้าทั่วไป พากลับไปหน้าแรกเพื่อช้อปปิ้งต่อ
      }

    } catch (error) {
      alert('เข้าสู่ระบบไม่สำเร็จ: อีเมลหรือรหัสผ่านไม่ถูกต้อง')
    }

  } else {
    // ==========================================
    // 🔵 โหมด REGISTER
    // ==========================================
    if (!form.value.username || !form.value.email || !form.value.password || !form.value.confirmPassword) {
      alert('กรุณากรอกข้อมูลให้ครบทุกช่อง')
      return
    }

    if (form.value.password !== form.value.confirmPassword) {
      alert('รหัสผ่านและการยืนยันรหัสผ่านไม่ตรงกัน!')
      return
    }

    try {
      // 💡 จุดเชื่อม Backend (Supabase):
      // const supabase = useSupabaseClient()
      // const { data, error } = await supabase.auth.signUp({
      //   email: form.value.email,
      //   password: form.value.password,
      //   options: { data: { username: form.value.username } } // บันทึกชื่อผู้ใช้ลง metadata
      // })
      // if (error) throw error

      console.log('สมัครสมาชิกสำเร็จ:', form.value.email)
      alert('สมัครสมาชิกสำเร็จ! กรุณาเข้าสู่ระบบ')
      
      // สลับกลับมาหน้า Login และล้างรหัสผ่านทิ้ง
      isLogin.value = true
      form.value.password = ''
      form.value.confirmPassword = ''

    } catch (error) {
      alert('สมัครสมาชิกไม่สำเร็จ: อาจมีอีเมลนี้ในระบบแล้ว')
    }
  }
}

// ฟังก์ชันสำหรับสลับโหมดและล้างข้อมูล
const toggleMode = () => {
  isLogin.value = !isLogin.value
  form.value = { username: '', email: '', password: '', confirmPassword: '' }
}
</script>

<template>
  <div class="min-h-screen bg-[#fff5f7] flex items-center justify-center py-12 px-4 sm:px-6 lg:px-8">
    
    <div class="max-w-md w-full bg-white rounded-[2rem] shadow-md p-8 md:p-10 space-y-8 transition-all duration-500">
      
      <div>
        <h2 class="text-center text-3xl font-extrabold text-gray-800">
          {{ isLogin ? 'Login' : 'Register' }}
        </h2>
        <p class="mt-2 text-center text-sm text-gray-500">
          {{ isLogin ? 'ยินดีต้อนรับกลับมา! กรุณาเข้าสู่ระบบ' : 'สร้างบัญชีใหม่เพื่อเริ่มช้อปปิ้งเลย' }}
        </p>
      </div>

      <form class="space-y-5" @submit.prevent="handleSubmit">
        
        <div v-if="!isLogin" class="transition-all duration-300">
          <label class="block text-sm font-medium text-gray-700 mb-1 pl-1">Username</label>
          <input 
            v-model="form.username" 
            type="text" 
            placeholder="ชื่อผู้ใช้งาน"
            class="w-full bg-gray-50 border border-gray-200 rounded-full px-5 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1 pl-1">Email</label>
          <input 
            v-model="form.email" 
            type="email" 
            placeholder="อีเมลของคุณ"
            required
            class="w-full bg-gray-50 border border-gray-200 rounded-full px-5 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
          />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1 pl-1">Password</label>
          <input 
            v-model="form.password" 
            type="password" 
            placeholder="รหัสผ่าน"
            required
            class="w-full bg-gray-50 border border-gray-200 rounded-full px-5 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
          />
        </div>

        <div v-if="!isLogin" class="transition-all duration-300">
          <label class="block text-sm font-medium text-gray-700 mb-1 pl-1">Confirm Password</label>
          <input 
            v-model="form.confirmPassword" 
            type="password" 
            placeholder="ยืนยันรหัสผ่านอีกครั้ง"
            class="w-full bg-gray-50 border border-gray-200 rounded-full px-5 py-3 focus:ring-2 focus:ring-[#ff8fa3] outline-none transition-all"
          />
        </div>

        <div class="pt-4">
          <button 
            type="submit" 
            class="w-full bg-[#ff8fa3] text-white text-lg font-bold py-3.5 rounded-full hover:bg-pink-400 hover:shadow-md transition-all duration-300"
          >
            {{ isLogin ? 'Sign In' : 'Sign Up' }}
          </button>
        </div>
      </form>

      <div class="text-center mt-6">
        <p class="text-sm text-gray-600">
          {{ isLogin ? "Don't have an account?" : "Already have an account?" }}
          <button 
            @click="toggleMode" 
            type="button"
            class="font-bold text-[#ff8fa3] hover:text-pink-500 hover:underline transition-colors ml-1 focus:outline-none"
          >
            {{ isLogin ? 'Sign Up' : 'Sign In' }}
          </button>
        </p>
      </div>

    </div>
  </div>
</template>