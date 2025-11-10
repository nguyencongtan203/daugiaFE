<template>
  <div
    v-if="show"
    class="fixed inset-0 bg-black/50 backdrop-blur-sm flex items-center justify-center z-[9999]"
  >
    <div
      class="relative w-full max-w-md bg-white rounded-2xl shadow-2xl p-8 border border-gray-200 transition-all"
    >
      <button
        @click="close"
        class="absolute top-3 right-3 text-gray-400 hover:text-gray-600 transition"
      >
        ✖
      </button>

      <h3 class="text-xl font-semibold text-gray-800 mb-6 text-center">Đổi mật khẩu</h3>

      <form @submit.prevent="handleChangePassword" class="space-y-4">
        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Mật khẩu cũ</label>
          <input type="password" v-model="matkhaucu" class="input" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1">Mật khẩu mới</label>
          <input type="password" v-model="matkhaumoi" class="input" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 mb-1"
            >Xác nhận mật khẩu</label
          >
          <input type="password" v-model="xacnhan" class="input" required />
        </div>

        <button type="submit" :disabled="loading" class="btn-primary">
          <span v-if="loading">Đang xử lý...</span>
          <span v-else>Đổi mật khẩu</span>
        </button>

        <p v-if="error" class="error-msg">{{ error }}</p>
      </form>
    </div>
  </div>

  <!-- Toast -->
  <transition name="slide-fade">
    <div
      v-if="toast.show"
      :class="[
        'fixed top-5 right-5 min-w-[250px] px-4 py-3 rounded-lg shadow-md text-gray-800 bg-white border-l-4 z-50',
        toast.type === 'success' ? 'border-green-500' : 'border-red-500',
      ]"
    >
      {{ toast.message }}
    </div>
  </transition>
</template>

<script setup>
import { ref, defineExpose, inject } from "vue";
import axios from "axios";
import Cookies from "js-cookie";
import { useUserStore } from "../stores/userStore";

const show = ref(false);
const matkhaucu = ref("");
const matkhaumoi = ref("");
const xacnhan = ref("");
const loading = ref(false);
const error = ref("");
const toast = ref({ show: false, message: "", type: "success" });
const authPopup = inject("authPopup");
const userStore = useUserStore();

// Mở popup
function open() {
  show.value = true;
  matkhaucu.value = "";
  matkhaumoi.value = "";
  xacnhan.value = "";
  error.value = "";
}

// Đóng popup
function close() {
  show.value = false;
}

// Toast
function showToast(message, type = "success") {
  toast.value = { show: true, message, type };
  setTimeout(() => (toast.value.show = false), 3000);
}

// Đổi mật khẩu
async function handleChangePassword() {
  error.value = "";
  if (matkhaumoi.value !== xacnhan.value) {
    error.value = "Mật khẩu mới và xác nhận không khớp!";
    return;
  }

  loading.value = true;
  try {
    const token = Cookies.get("jwt_token");
    if (!token) {
      error.value = "Phiên đăng nhập hết hạn. Vui lòng đăng nhập lại.";
      return;
    }

    const payload = {
      matkhaucu: matkhaucu.value,
      matkhaumoi: matkhaumoi.value,
    };

    const res = await axios.put(
      "http://localhost:8082/api/users/change-password",
      payload,
      { headers: { Authorization: `Bearer ${token}` } }
    );

    if (res.data.code === 200) {
      showToast(res.data.message, "success");
      // Xóa token và logout
      Cookies.remove("jwt_token");
      userStore.logout();
      close();
      authPopup?.value?.open("login");
    } else {
      error.value = res.data.message || "Đổi mật khẩu thất bại!";
    }
  } catch (err) {
    console.error(err);
    error.value = "Lỗi kết nối tới máy chủ!";
  } finally {
    loading.value = false;
  }
}

// Expose
defineExpose({ open, close });
</script>
<style scoped>
@import "@/assets/styles/toast.css";
@import "@/assets/styles/auth.css";
</style>
