<script setup>
import { ref, onMounted } from 'vue'

definePageMeta({ layout: 'admin' })

const route = useRoute()
const orderId = route.params.id

// [BACKEND: การเตรียมตัวเชื่อมต่อฐานข้อมูล]
const supabase = useSupabaseClient()

const order = ref(null)
const isLoading = ref(true)

// [BACKEND: ดึงข้อมูล Order แบบเจาะลึก (Deep Fetch)]
const fetchOrderDetail = async () => {
  isLoading.value = true
  //Mock Data สำหรับหน้า Detail (โครงสร้างตรงกับ Supabase Join)
  setTimeout(() => {
    // 💡 ข้อมูลจำลองสำหรับหน้า Detail (โครงสร้างตรงกับ Supabase Join)
    order.value = {
      id: orderId, // ใช้ ID จาก URL ปัจจุบัน
      order_no: 'ORD-2603-002',
      status: 'pending_review', // ลองเปลี่ยนสถานะดูเพื่อให้ปุ่ม UI เปลี่ยน
      total_amount: 850,
      created_at: '2026-03-18T14:30:00Z',
      order_source: 'web',
      members: {
        full_name: 'วิภาดา รักงานคราฟต์',
        phone: '081-234-5678',
        email: 'wipada@example.com'
      },
      order_items: [
        {
          id: 'item-1', qty: 2, unit_price: 250, line_total: 500, product_id: 'prod-1',
          products: { name: 'กระเป๋าถักไหมพรมสไตล์มินิมอล', sku: 'BAG-001', image_url: 'https://via.placeholder.com/150/ff8fa3/ffffff?text=Bag' }
        },
        {
          id: 'item-2', qty: 1, unit_price: 350, line_total: 350, product_id: 'prod-2',
          products: { name: 'พวงกุญแจดอกทิวลิป', sku: 'KEY-002', image_url: 'https://via.placeholder.com/150/a2d2ff/ffffff?text=Key' }
        }
      ],
      payments: [
        {
          id: 'pay-1',
          payment_method: 'bank_transfer',
          payment_status: 'pending',
          payment_amount: 850,
          slip_image_url: 'https://via.placeholder.com/300x500/e2e8f0/64748b?text=Slip+Image+Mockup'
        }
      ]
    }
    isLoading.value = false
  }, 800)

  /*
  try {
    // ใช้ Supabase ดึงข้อมูลหลัก พร้อม Join ตารางย่อยทั้งหมดที่เกี่ยวข้อง
    const { data, error } = await supabase
      .from('orders')
      .select(`
        *,
        members (*),
        order_items (
          *,
          products (name, sku, image_url)
        ),
        payments (*)
      `)
      .eq('id', orderId)
      .single() // บังคับดึงมาแค่ 1 แถว

    if (error) throw error
    order.value = data
  } catch (err) {
    console.error('Error fetching order details:', err)
  } finally {
    isLoading.value = false
  }
    */
}


// [BACKEND: ฟังก์ชันอัปเดตสถานะออเดอร์]
const updateOrderStatus = async (newStatus) => {
  if(!confirm(`ยืนยันการเปลี่ยนสถานะเป็น "${statusLabels[newStatus]}"?`)) return
  
  try {
    const { error } = await supabase
      .from('orders')
      .update({ status: newStatus })
      .eq('id', orderId)

    if (error) throw error
    order.value.status = newStatus // อัปเดต UI
    alert('อัปเดตสถานะเรียบร้อย')
  } catch (err) {
    alert('เกิดข้อผิดพลาดในการอัปเดต')
    console.error(err)
  }
}

// [BACKEND: ฟังก์ชันอนุมัติสลิปชำระเงิน]
const approvePayment = async () => {
  try {
    // 1. อัปเดตตาราง Payment
    if(order.value.payments && order.value.payments.length > 0) {
       await supabase.from('payments')
        .update({ payment_status: 'completed' }) // สมมติว่าสร้างฟิลด์ payment_status ไว้
        .eq('order_id', orderId)
    }
    // 2. เปลี่ยนสถานะออเดอร์เป็น Paid
    await updateOrderStatus('paid')
  } catch (err) {
    console.error('Approve failed:', err)
  }
}

const statusColors = {
  pending_payment: 'bg-yellow-100 text-yellow-800',
  pending_review: 'bg-orange-100 text-orange-800',
  paid: 'bg-green-100 text-green-800',
  shipped: 'bg-blue-100 text-blue-800',
  cancelled: 'bg-red-100 text-red-800'
}
const statusLabels = { pending_payment: 'รอชำระเงิน', pending_review: 'รอตรวจสอบสลิป', paid: 'ชำระเงินแล้ว', shipped: 'จัดส่งแล้ว', cancelled: 'ยกเลิก' }

onMounted(() => {
  fetchOrderDetail()
})

// [BACKEND: ฟังก์ชันยกเลิกออเดอร์ และ คืนสต๊อกสินค้า]
const cancelOrder = async () => {
  if(!confirm(`ยืนยันการยกเลิกคำสั่งซื้อ ${order.value.order_no} และคืนสต๊อกสินค้าหรือไม่?`)) return
  
  isLoading.value = true
  try {
    // 1. อัปเดตสถานะออเดอร์เป็น 'cancelled'
    const { error: orderError } = await supabase
      .from('orders')
      .update({ status: 'cancelled' })
      .eq('id', orderId)
    if (orderError) throw orderError

    // 2. วนลูปคืนสต๊อกสินค้า และบันทึกประวัติความเคลื่อนไหว (Stock Movement)
    if (order.value.order_items && order.value.order_items.length > 0) {
      for (const item of order.value.order_items) {
        
        // ก. ดึงจำนวนสต๊อกปัจจุบันของสินค้านั้นๆ
        const { data: productData } = await supabase
          .from('products')
          .select('stock_qty')
          .eq('id', item.product_id)
          .single()
        
        const currentStock = productData ? Number(productData.stock_qty) : 0
        const newStock = currentStock + Number(item.qty) // บวกของคืนกลับเข้าไป

        // ข. อัปเดตสต๊อกกลับเข้าตาราง products
        await supabase
          .from('products')
          .update({ stock_qty: newStock })
          .eq('id', item.product_id)

        // ค. บันทึกประวัติลง stock_movements (สมมติใช้ type 'adjust' หรือถ้าคุณเพิ่ม 'sale_return' ไว้ก็ใช้ได้ครับ)
        await supabase
          .from('stock_movements')
          .insert({
            item_type: 'product',
            movement_type: 'adjust', 
            ref_table: 'orders',
            ref_id: orderId,
            product_id: item.product_id,
            qty_in: item.qty, // รับของกลับเข้า
            qty_out: 0,
            balance_after: newStock
          })
      }
    }

    
  // 3. อัปเดต UI ให้เป็นปัจจุบัน
    order.value.status = 'cancelled'
    alert('ยกเลิกออเดอร์และคืนสต๊อกสินค้าเข้าคลังเรียบร้อยแล้ว')

  } catch (err) {
    alert('เกิดข้อผิดพลาดในการยกเลิกออเดอร์')
    console.error('Cancel order error:', err)
  } finally {
    isLoading.value = false
  }
}
</script>

<template>
  <div class="p-6 max-w-5xl mx-auto">
    <div v-if="isLoading" class="text-center py-20 text-gray-500">กำลังโหลดข้อมูล...</div>
    
    <div v-else-if="order">
      <!-- [UI: ส่วนหัวเรื่อง ปุ่มย้อนกลับ และ Badge สถานะ] -->
      <div class="flex justify-between items-center mb-6">
        <div class="flex items-center gap-4">
          <NuxtLink to="/admin/order" class="p-2 bg-gray-100 rounded-lg hover:bg-gray-200 transition">
            <svg class="w-5 h-5 text-gray-600" fill="none" viewBox="0 0 24 24" stroke="currentColor"><path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M10 19l-7-7m0 0l7-7m-7 7h18" /></svg>
          </NuxtLink>
          <div>
            <h1 class="text-2xl font-bold text-gray-800">ออเดอร์ {{ order.order_no }}</h1>
            <p class="text-sm text-gray-500">{{ new Date(order.created_at).toLocaleString('th-TH') }}</p>
          </div>
        </div>
        <span :class="['px-4 py-1.5 rounded-full text-sm font-bold', statusColors[order.status]]">
          {{ statusLabels[order.status] }}
        </span>
      </div>

      <!-- [UI: โครงสร้าง Grid ซ้าย-ขวา] -->
      <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
        
        <!-- คอลัมน์ซ้าย: รายการสินค้า -->
        <div class="lg:col-span-2 space-y-6">
          <div class="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
            <h2 class="text-lg font-bold mb-4 border-b pb-2">รายการสินค้า</h2>
            <div class="space-y-4">
              <!-- วนลูป order_items ที่ได้จาก Backend -->
              <div v-for="item in order.order_items" :key="item.id" class="flex justify-between items-center">
                <div class="flex items-center gap-4">
                  <div class="w-16 h-16 bg-gray-100 rounded-lg overflow-hidden">
                    <img v-if="item.products.image_url" :src="item.products.image_url" class="w-full h-full object-cover"/>
                  </div>
                  <div>
                    <p class="font-medium text-gray-800">{{ item.products.name }}</p>
                    <p class="text-xs text-gray-500">SKU: {{ item.products.sku }} | ฿{{ item.unit_price }} x {{ item.qty }}</p>
                  </div>
                </div>
                <p class="font-bold text-gray-800">฿{{ item.line_total.toLocaleString() }}</p>
              </div>
            </div>
            <!-- สรุปยอด -->
            <div class="mt-6 pt-4 border-t border-gray-100 flex justify-between items-center">
              <p class="font-bold text-gray-600">ยอดรวมทั้งสิ้น</p>
              <p class="text-xl font-black text-[#ff8fa3]">฿{{ order.total_amount.toLocaleString() }}</p>
            </div>
          </div>
        </div>

        <!-- คอลัมน์ขวา: ข้อมูลลูกค้าและการจัดการ -->
        <div class="space-y-6">
          <!-- ข้อมูลลูกค้า -->
          <div class="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
            <h2 class="text-lg font-bold mb-4 border-b pb-2">ข้อมูลลูกค้า</h2>
            <div v-if="order.members">
              <p class="font-medium text-gray-800">{{ order.members.full_name }}</p>
              <p class="text-sm text-gray-600 mt-1">📞 {{ order.members.phone || 'ไม่ได้ระบุ' }}</p>
              <p class="text-sm text-gray-600 mt-1">📧 {{ order.members.email || 'ไม่ได้ระบุ' }}</p>
            </div>
            <div v-else>
              <p class="text-gray-500">ลูกค้า Walk-in / ทั่วไป</p>
            </div>
          </div>

          <!-- ข้อมูลการชำระเงิน & ปุ่ม Action -->
          <div class="bg-white p-6 rounded-xl shadow-sm border border-gray-100">
            <h2 class="text-lg font-bold mb-4 border-b pb-2">การชำระเงิน</h2>
            
            <!-- ดึงรูปสลิปจาก Array payments (ถ้ามีข้อมูล) -->
            <div v-if="order.payments && order.payments.length > 0" class="mb-4">
              <p class="text-sm font-medium mb-2">สลิปโอนเงิน:</p>
              <img :src="order.payments[0].slip_image_url" alt="Slip" class="w-full rounded border border-gray-200 cursor-pointer hover:opacity-90" v-if="order.payments[0].slip_image_url"/>
              <p v-else class="text-sm text-gray-500">ไม่มีรูปสลิป</p>
            </div>

            <!-- [UI & LOGIC: ปุ่มจัดการตามสถานะปัจจุบัน] -->
            <div class="space-y-2 mt-4">
              <button 
                v-if="order.status === 'pending_review'" 
                @click="approvePayment"
                class="w-full py-2 bg-green-500 text-white font-bold rounded-lg hover:bg-green-600 transition"
              >
                ✅ ตรวจสอบสลิปถูกต้อง (รับยอด)
              </button>

              <button 
                v-if="order.status === 'paid'" 
                @click="updateOrderStatus('shipped')"
                class="w-full py-2 bg-blue-500 text-white font-bold rounded-lg hover:bg-blue-600 transition"
              >
                📦 ยืนยันการจัดส่งแล้ว
              </button>

              <button 
                v-if="order.status !== 'cancelled' && order.status !== 'shipped'" 
                @click="updateOrderStatus('cancelOrder')"
                class="w-full py-2 bg-white border border-red-500 text-red-500 font-bold rounded-lg hover:bg-red-50 transition"
              >
                ❌ ยกเลิกคำสั่งซื้อ
              </button>
            </div>
          </div>

        </div>
      </div>
    </div>
  </div>
</template>