<script setup>
// [ไม่มี Backend]
// หน้านี้ถูกปรับให้แสดงเฉพาะข้อมูลการติดต่อโดยตรง (แผนที่, เบอร์โทร, โซเชียลมีเดีย) 
// จึงไม่ต้องใช้ตัวแปรหรือฟังก์ชันสำหรับจัดการฟอร์มอีกต่อไปครับ
</script>

<template>
  <div class="min-h-screen bg-[#f8f9fa] py-10 px-4 sm:px-6 lg:px-8">
    
    <!-- กล่อง Content หลัก (จัดให้อยู่กึ่งกลางหน้าจอเสมอ) -->
    <div class="max-w-4xl mx-auto flex flex-col gap-10">
      
      <!-- 1. ส่วนแผนที่ (Google Maps) -->
      <!-- [Responsive]: ปรับสัดส่วนให้พอดีจอ มือถือ 4:3, คอมพิวเตอร์ 16:9 -->
      <div class="w-full aspect-[4/3] md:aspect-[21/9] bg-gray-200 overflow-hidden shadow-sm">
        <iframe 
          src="https://www.google.com/maps/embed?pb=!1m18!1m12!1m3!1d3826.680695027591!2d102.8236111!3d16.4427778!2m3!1f0!2f0!3f0!3m2!1i1024!2i768!4f13.1!3m3!1m2!1s0x31222a2555555555%3A0x1111111111111111!2sCollege%20of%20Computing%2C%20Khon%20Kaen%20University!5e0!3m2!1sen!2sth!4v1700000000000!5m2!1sen!2sth" 
          width="100%" 
          height="100%" 
          style="border:0;" 
          allowfullscreen="" 
          loading="lazy" 
          referrerpolicy="no-referrer-when-downgrade">
        </iframe>
      </div>

      <!-- 2. ส่วนข้อมูลที่อยู่และโซเชียล -->
      <!-- จัดให้อยู่กึ่งกลางหน้าจอ แต่เนื้อหาด้านในเรียงชิดซ้ายให้อ่านง่าย -->
      <div class="flex flex-col items-center">
        <div class="flex flex-col gap-8 w-full max-w-2xl px-2 md:px-0">
          
          <!-- ที่อยู่ -->
          <div>
            <h3 class="text-xl md:text-2xl font-medium text-gray-800 mb-2">
              วิทยาลัยการคอมพิวเตอร์ มหาวิทยาลัยขอนแก่น
            </h3>
            <p class="text-gray-800 leading-relaxed text-base md:text-lg">
              123 อาคารวิทยวิภาส ถ.มิตรภาพ ต.ในเมือง<br>
              อ.เมืองขอนแก่น จ.ขอนแก่น 40002
            </p>
          </div>

          <!-- ช่องทางการติดต่อ (Icons & Text) -->
          <div class="flex flex-col gap-5 mt-2">
            
            <!-- เบอร์โทรศัพท์ -->
            <div class="flex items-center gap-4">
              <div class="w-12 h-12 rounded-full border-[3px] border-[#1d4ed8] flex items-center justify-center text-[#1d4ed8]">
                <svg xmlns="http://www.w3.org/2000/svg" class="h-6 w-6" viewBox="0 0 20 20" fill="currentColor">
                  <path d="M2 3a1 1 0 011-1h2.153a1 1 0 01.986.836l.74 4.435a1 1 0 01-.54 1.06l-1.548.773a11.037 11.037 0 006.105 6.105l.774-1.548a1 1 0 011.059-.54l4.435.74a1 1 0 01.836.986V17a1 1 0 01-1 1h-2C7.82 18 2 12.18 2 5V3z" />
                </svg>
              </div>
              <span class="text-gray-800 text-lg md:text-xl font-medium">043 009 700 ต่อ 44463, 50525</span>
            </div>

            <!-- IG (Instagram) -->
            <div class="flex items-center gap-4">
              <!-- สร้างพื้นหลังสี IG ด้วย Gradient ของ Tailwind -->
              <div class="w-12 h-12 rounded-xl bg-gradient-to-tr from-[#facc15] via-[#ec4899] to-[#8b5cf6] flex items-center justify-center text-white p-2.5">
                <svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round" stroke-linejoin="round" class="w-full h-full">
                  <rect x="2" y="2" width="20" height="20" rx="5" ry="5"></rect>
                  <path d="M16 11.37A4 4 0 1 1 12.63 8 4 4 0 0 1 16 11.37z"></path>
                  <line x1="17.5" y1="6.5" x2="17.51" y2="6.5"></line>
                </svg>
              </div>
              <span class="text-gray-800 text-lg md:text-xl font-medium">IG : 123456</span>
            </div>

            <!-- LINE -->
            <div class="flex items-center gap-4">
              <!-- สีเขียวของ LINE คือ #06C755 -->
              <div class="w-12 h-12 rounded-xl bg-[#06C755] flex items-center justify-center text-white font-black text-sm tracking-wider">
                LINE
              </div>
              <span class="text-gray-800 text-lg md:text-xl font-medium">LineID : 123456</span>
            </div>

          </div>

        </div>
      </div>

    </div>
  </div>
</template>