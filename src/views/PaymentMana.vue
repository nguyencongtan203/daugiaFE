<template>
  <div class="p-6 max-w-4xl mx-auto">
    <!-- Tabs -->
    <div class="menu flex gap-4 mb-4 border-b pb-2">
      <button
        :class="{ active: activeTab === 'deposit' }"
        @click="activeTab = 'deposit'"
        class="px-4 py-2 rounded"
      >
        Thanh toán tiền cọc
      </button>
      <button
        :class="{ active: activeTab === 'win' }"
        @click="activeTab = 'win'"
        class="px-4 py-2 rounded"
      >
        Thanh toán thắng phiên
      </button>
    </div>

    <!-- TAB THANH TOÁN TIỀN CỌC -->
    <div v-if="activeTab === 'deposit'">
      <h2 class="text-xl font-semibold mb-4">Phiếu thanh toán tiền cọc</h2>

      <div v-if="loading">Đang tải...</div>
      <div v-else-if="error" class="text-red-500">{{ error }}</div>
      <div v-else-if="deposits.length === 0">
        <p>Không có phiếu thanh toán nào.</p>
      </div>
      <div v-else class="space-y-4">
        <div
          v-for="item in deposits"
          :key="item.matc"
          class="border rounded-lg p-4 shadow-sm bg-white"
        >
          <p><b>Mã phiếu:</b> {{ item.matc }}</p>
          <p><b>Phiên đấu giá:</b> {{ item.phienDauGia.maphiendg }}</p>
          <p><b>Tiền cọc:</b> {{ formatCurrency(item.phienDauGia.tiencoc) }}</p>
          <p><b>Hạn thanh toán:</b> {{ formatDate(item.thoigianthanhtoan) }}</p>
          <p><b>Trạng thái:</b> {{ item.trangthai }}</p>

          <button
            v-if="item.trangthai === 'Chưa thanh toán'"
            @click="handlePay(item)"
            class="mt-3 bg-[#22b8cf] text-white px-6 py-2 rounded-lg hover:bg-[#1098ad]"
          >
            Thanh toán
          </button>
        </div>
      </div>
    </div>

    <!-- TAB THANH TOÁN THẮNG PHIÊN (sau này bổ sung) -->
    <div v-if="activeTab === 'win'">
      <h2 class="text-xl font-semibold mb-4">Thanh toán thắng phiên</h2>
      <p class="text-slate-600">Tính năng này sẽ được cập nhật sau.</p>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Cookies from "js-cookie";

const deposits = ref([]);
const loading = ref(true);
const error = ref(null);
const activeTab = ref("deposit");

const formatCurrency = (n) =>
  new Intl.NumberFormat("vi-VN", {
    style: "currency",
    currency: "VND",
  }).format(n || 0);

const formatDate = (iso) => {
  if (!iso) return "-";
  const d = new Date(iso);
  return d.toLocaleString("vi-VN", {
    year: "numeric",
    month: "2-digit",
    day: "2-digit",
    hour: "2-digit",
    minute: "2-digit",
    hour12: false,
  });
};

onMounted(fetchDeposits);

async function fetchDeposits() {
  loading.value = true;
  error.value = null;
  try {
    const token = Cookies.get("jwt_token");
    if (!token) {
      error.value = "Bạn cần đăng nhập để xem phiếu thanh toán.";
      return;
    }

    const res = await axios.get("http://localhost:8082/api/deposit-payments/find-by-user", {
      headers: { Authorization: `Bearer ${token}` },
    });

    if (res.data.code === 200) {
      deposits.value = res.data.result || [];
    } else {
      error.value = res.data.message;
    }
  } catch (err) {
    error.value = "Lỗi khi tải dữ liệu: " + err.message;
  } finally {
    loading.value = false;
  }
}

async function handlePay(item) {
  try {
    const res = await axios.get(
      `http://localhost:8082/api/deposit-payments/create-order?matc=${item.matc}&amount=${item.phienDauGia.tiencoc || 0}`
    );
    if (res.data.code === 200) {
      window.location.href = res.data.result; // ✅ chuyển tới trang thanh toán VNPAY
    } else {
      alert(res.data.message);
    }
  } catch (err) {
    alert("Lỗi khi tạo URL thanh toán: " + err.message);
  }
}
</script>

<style scoped>
.active {
  border-bottom: 2px solid #22b8cf;
  color: #22b8cf;
  font-weight: 600;
}
</style>
