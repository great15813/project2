<script setup>
import { ref, onMounted } from 'vue'

// [BACKEND: ส่วนนี้คือการจำลองข้อมูลที่ดึงมาจาก Database (Table: faqs)]
// const { fetchFaqs } = useFaqs() // สมมติว่ามี Composable ดึงข้อมูล
const faqs = ref([
  {
    id: 1,
    question: 'จัดส่งสินค้าอย่างไร',
    answer: 'การจัดส่งสินค้าของเรา<br>• ส่งผ่านไปรษณีย์ไทย EMS (ในประเทศ)<br>• ส่งผ่านไปรษณีย์ไทย ePacket (ต่างประเทศ)'
  },
  {
    id: 2,
    question: 'ใช้เวลาจัดส่งกี่วัน?',
    answer: 'หลังจากยืนยันการชำระเงิน เราจะจัดส่งภายใน 1-2 วันทำการครับ<br>• กรุงเทพฯ และปริมณฑล 1-2 วัน<br>• ต่างจังหวัด 2-3 วัน'
  },
  {
    id: 3,
    question: 'สามารถเปลี่ยนหรือคืนสินค้าได้ไหม?',
    answer: 'รับเปลี่ยนหรือคืนสินค้าภายใน 7 วัน เฉพาะกรณีที่สินค้าชำรุดจากการขนส่ง หรือทางร้านส่งสินค้าผิดแบบเท่านั้นครับ'
  }
])

// ตัวแปรสำหรับเก็บว่าตอนนี้เปิดคำถามข้อไหนอยู่ (null คือปิดหมด)
const activeFaq = ref(null)

// ฟังก์ชันสลับเปิด/ปิด Accordion
const toggleFaq = (id) => {
  if (activeFaq.value === id) {
    activeFaq.value = null // ถ้ากดซ้ำข้อเดิมให้ปิด
  } else {
    activeFaq.value = id // เปิดข้อที่กด
  }
}

// ดึงข้อมูลจริงจาก Backend ตอนเปิดหน้าเว็บ
/*
onMounted(async () => {
  const data = await fetchFaqs()
  faqs.value = data
})
*/
</script>

<template>
  <div class="min-h-screen bg-white py-8 px-4 sm:px-6 lg:px-8">
    
    <!-- ใช้ flex-col บนมือถือ และ flex-row บนจอใหญ่ (lg) -->
    <div class="max-w-7xl mx-auto flex flex-col lg:flex-row gap-8 lg:gap-12 items-start">
      
      <!-- ฝั่งซ้าย: รายการคำถาม FAQ (กว้าง 100% บนมือถือ, กว้าง 60% บนจอใหญ่) -->
      <div class="w-full lg:w-3/5 flex flex-col gap-4">
        
        <!-- วนลูปสร้างกล่องคำถามจาก Backend -->
        <div 
          v-for="faq in faqs" 
          :key="faq.id"
          class="bg-[#fff0f3] rounded-2xl overflow-hidden transition-all duration-300"
        >
          <!-- ส่วนหัวคำถาม (คลิกเพื่อเปิด/ปิดได้) -->
          <button 
            @click="toggleFaq(faq.id)"
            class="w-full px-6 py-5 text-left flex justify-between items-center focus:outline-none"
          >
            <span class="font-medium text-gray-800">
              Question : {{ faq.question }}
            </span>
            <!-- ไอคอนลูกศร ชี้ลงตอนปิด ชี้ขึ้นตอนเปิด -->
            <svg 
              xmlns="http://www.w3.org/2000/svg" 
              class="h-5 w-5 text-gray-500 transform transition-transform duration-300" 
              :class="{ 'rotate-180': activeFaq === faq.id }"
              fill="none" viewBox="0 0 24 24" stroke="currentColor"
            >
              <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M19 9l-7 7-7-7" />
            </svg>
          </button>

          <!-- ส่วนคำตอบ (แสดงเมื่อ activeFaq ตรงกับ id ของข้อนี้) -->
          <!-- ใช้ v-html เพื่อเรนเดอร์แท็ก <br> ที่เก็บมาจาก Database -->
          <div 
            v-show="activeFaq === faq.id"
            class="px-6 pb-5 pt-2 border-t border-pink-200/50 mx-4"
          >
            <span class="font-medium text-gray-800 block mb-2">Answer :</span>
            <div class="text-gray-600 leading-relaxed pl-4" v-html="faq.answer"></div>
          </div>
        </div>

      </div>

      <!-- ฝั่งขวา: รูปภาพตกแต่ง (จะย้ายมาอยู่ด้านล่างสุดบนมือถือ) -->
      <div class="w-full lg:w-2/5">
        <div class="bg-gray-100 rounded-3xl overflow-hidden aspect-[4/5] sticky top-24">
          <img 
            src="......." 
            alt="Product Decoration" 
            class="w-full h-full object-cover"
          />
        </div>
      </div>

    </div>
  </div>
</template>