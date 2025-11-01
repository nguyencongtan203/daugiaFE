<template>
  <div class="min-h-screen flex items-center justify-center gradient-custom-2">
    <div
      class="w-full max-w-md bg-white/95 rounded-2xl shadow-xl p-8 border border-gray-100 backdrop-blur-sm"
    >
      <!-- Tiêu đề -->
      <h2 class="text-2xl font-bold text-center text-gray-800 mb-6 tracking-wide">
        Đăng nhập hệ thống
      </h2>

      <!-- Form đăng nhập -->
      <form @submit.prevent="handleLogin" class="space-y-5">
        <!-- Email -->
        <div>
          <label for="email" class="block text-sm font-medium text-gray-700 mb-1">
            Email
          </label>
          <input
            id="email"
            v-model="email"
            type="email"
            required
            placeholder="Nhập email của bạn"
            class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition"
          />
        </div>

        <!-- Mật khẩu -->
        <div>
          <label for="password" class="block text-sm font-medium text-gray-700 mb-1">
            Mật khẩu
          </label>
          <div class="relative">
            <input
              id="password"
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              required
              placeholder="Nhập mật khẩu"
              class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-indigo-500 focus:border-indigo-500 outline-none transition pr-10"
            />

            <!-- Nút hiện/ẩn mật khẩu -->
            <button
              type="button"
              @click="togglePassword"
              class="absolute inset-y-0 right-0 flex items-center pr-3 text-gray-500 hover:text-indigo-600 transition"
            >
              <svg
                v-if="showPassword"
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke-width="1.8"
                stroke="currentColor"
                class="w-5 h-5"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M2.458 12C3.732 7.943 7.523 5 12 5c4.478 0 8.268 2.943 9.542 7-1.274 4.057-5.064 7-9.542 7-4.477 0-8.268-2.943-9.542-7z"
                />
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
                />
              </svg>

              <svg
                v-else
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 24 24"
                stroke-width="1.8"
                stroke="currentColor"
                class="w-5 h-5"
              >
                <path
                  stroke-linecap="round"
                  stroke-linejoin="round"
                  d="M3.98 8.223a10.477 10.477 0 00-1.518 3.777C3.736 16.057 7.523 19 12 19c1.829 0 3.55-.43 5.06-1.198M21 21L3 3m4.06 4.06A9.967 9.967 0 0112 5c4.478 0 8.264 2.943 9.54 7a10.478 10.478 0 01-1.314 2.771M9.88 9.88A3 3 0 0114.12 14.12"
                />
              </svg>
            </button>
          </div>
        </div>

        <!-- Nút đăng nhập (sử dụng gradient-custom-2) -->
        <button
          type="submit"
          :disabled="loading"
          class="w-full py-2.5 text-white font-semibold rounded-lg transition gradient-custom-2 hover:opacity-90 disabled:opacity-50 disabled:cursor-not-allowed shadow-md"
        >
          <span v-if="loading" class="flex items-center justify-center gap-2">
            <svg
              class="w-5 h-5 animate-spin text-white"
              xmlns="http://www.w3.org/2000/svg"
              fill="none"
              viewBox="0 0 24 24"
            >
              <circle
                class="opacity-25"
                cx="12"
                cy="12"
                r="10"
                stroke="currentColor"
                stroke-width="4"
              ></circle>
              <path
                class="opacity-75"
                fill="currentColor"
                d="M4 12a8 8 0 018-8v4a4 4 0 00-4 4H4z"
              ></path>
            </svg>
            Đang xử lý...
          </span>
          <span v-else>Đăng nhập</span>
        </button>
      </form>

      <!-- Gợi ý thêm -->
      <div class="text-center mt-5 text-sm text-gray-600">
        <a href="#" class="text-indigo-600 hover:underline">Quên mật khẩu?</a>
        <span class="mx-2">•</span>
        <a href="#" class="text-indigo-600 hover:underline">Đăng ký tài khoản mới</a>
      </div>

      <!-- Thông báo lỗi -->
      <p
        v-if="error"
        class="text-red-500 font-semibold text-center mt-4 bg-red-50 border border-red-200 rounded-lg py-2"
      >
        {{ error }}
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";
import Cookies from "js-cookie";
import { useRouter } from "vue-router";
import { useUserStore } from "../stores/userStore";

const email = ref("");
const password = ref("");
const showPassword = ref(false);
const loading = ref(false);
const error = ref("");
const router = useRouter();
const userStore = useUserStore();

const togglePassword = () => (showPassword.value = !showPassword.value);

const handleLogin = async () => {
  error.value = "";
  loading.value = true;

  try {
    const res = await axios.post("http://localhost:8082/api/auth/login", {
      email: email.value,
      matkhau: password.value,
    });

    if (res.data.code === 200) {
      const token = res.data.result;
      userStore.setToken(token);
      await userStore.fetchUser();
      alert("✅ Đăng nhập thành công!");
      router.push("/home");
    } else {
      error.value = res.data.message || "Đăng nhập thất bại!";
    }
  } catch (err) {
    console.error(err);
    error.value = "❌ Lỗi kết nối đến máy chủ!";
  } finally {
    loading.value = false;
  }
};
</script>
