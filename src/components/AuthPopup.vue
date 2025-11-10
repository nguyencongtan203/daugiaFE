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

      <!-- Tabs Đăng nhập / Đăng ký -->
      <div class="flex w-full mb-6 border-b border-gray-200">
        <button
          :class="[
            'w-1/2 text-center pb-2 border-b-2 text-lg font-semibold transition',
            mode === 'login'
              ? 'border-sky-500 text-sky-600'
              : 'border-transparent text-gray-500 hover:text-gray-700',
          ]"
          @click="mode = 'login'"
        >
          Đăng nhập
        </button>
        <button
          :class="[
            'w-1/2 text-center pb-2 border-b-2 text-lg font-semibold transition',
            mode === 'register'
              ? 'border-sky-500 text-sky-600'
              : 'border-transparent text-gray-500 hover:text-gray-700',
          ]"
          @click="mode = 'register'"
        >
          Đăng ký
        </button>
      </div>

      <!-- Form đăng nhập -->
      <form v-if="mode === 'login'" @submit.prevent="handleLogin" class="space-y-5">
        <div>
          <label for="email" class="block text-sm font-medium text-gray-700 mb-1"
            >Email</label
          >
          <input
            id="email"
            v-model="email"
            type="email"
            required
            placeholder="Nhập email của bạn"
            class="input"
          />
        </div>

        <div>
          <label for="password" class="block text-sm font-medium text-gray-700 mb-1"
            >Mật khẩu</label
          >
          <div class="relative">
            <input
              id="password"
              v-model="password"
              :type="showPassword ? 'text' : 'password'"
              required
              placeholder="Nhập mật khẩu"
              class="input pr-10"
            />
            <button
              type="button"
              @click="togglePassword"
              class="absolute inset-y-0 right-0 flex items-center pr-3 text-gray-500 hover:text-sky-600 transition"
            >
              <svg
                v-if="showPassword"
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 18 18"
                width="20"
                height="20"
                stroke="currentColor"
                stroke-width="0.1"
                class="w-5 h-5"
                space="preserve"
                style="
                  fill-rule: evenodd;
                  clip-rule: evenodd;
                  stroke-linejoin: round;
                  stroke-miterlimit: 2;
                "
              >
                <path
                  d="M16.666 6.734a.75.75 0 0 0-1.061.011A9.18 9.18 0 0 1 9 9.519a9.18 9.18 0 0 1-6.605-2.774.75.75 0 1 0-1.072 1.05c.515.525 1.074.99 1.669 1.393l-.881 1.441a.75.75 0 1 0 1.28.782l.906-1.482q1.019.497 2.124.769l-.333 1.655a.75.75 0 0 0 1.47.296l.339-1.685a11 11 0 0 0 2.206 0l.339 1.685a.75.75 0 0 0 1.47-.296l-.333-1.655a10.6 10.6 0 0 0 2.124-.769l.906 1.482a.75.75 0 1 0 1.28-.782l-.881-1.441a11 11 0 0 0 1.669-1.393.75.75 0 0 0-.011-1.061"
                  style="fill: currentcolor; fill-rule: nonzero"
                ></path>
              </svg>
              <svg
                v-else
                xmlns="http://www.w3.org/2000/svg"
                fill="none"
                viewBox="0 0 18 18"
                width="20"
                height="20"
                stroke="currentColor"
                stroke-width="0.1"
                class="w-5 h-5"
                space="preserve"
                style="
                  fill-rule: evenodd;
                  clip-rule: evenodd;
                  stroke-linejoin: round;
                  stroke-miterlimit: 2;
                "
              >
                <path
                  d="m15.008 6.083.881-1.441a.75.75 0 0 0-1.279-.783l-.907 1.482a10.6 10.6 0 0 0-2.124-.769l.333-1.655a.75.75 0 0 0-1.47-.296l-.339 1.685a11 11 0 0 0-2.206 0l-.339-1.685a.75.75 0 1 0-1.47.296l.333 1.655q-1.106.272-2.124.769L3.39 3.859a.75.75 0 0 0-1.279.783l.881 1.441c-.594.402-1.154.867-1.668 1.392a.75.75 0 1 0 1.072 1.05 9.18 9.18 0 0 1 6.605-2.774 9.18 9.18 0 0 1 6.605 2.774.747.747 0 0 0 1.061.011.75.75 0 0 0 .011-1.061 11 11 0 0 0-1.668-1.392z"
                  style="fill: currentcolor; fill-rule: nonzero"
                ></path>
                <path
                  d="M9 14a3.5 3.5 0 1 0 0-7 3.5 3.5 0 0 0 0 7"
                  style="fill: currentcolor; fill-rule: nonzero"
                ></path>
              </svg>
            </button>
          </div>
        </div>

        <div class="text-right">
          <a
            href="#"
            class="text-sm text-sky-600 hover:text-sky-700 font-medium"
            @click.prevent="handleForgotPassword"
          >
            Quên mật khẩu?
          </a>
        </div>
        <button type="submit" :disabled="loading" class="btn-primary">
          <span v-if="loading">Đang xử lý...</span>
          <span v-else>Đăng nhập</span>
        </button>

        <p v-if="error" class="error-msg">{{ error }}</p>
      </form>

      <!-- Form đăng ký -->
      <form v-else @submit.prevent="handleRegister" class="space-y-5">
        <div class="grid grid-cols-2 gap-4">
          <div>
            <label for="registerHo" class="block text-sm font-medium text-gray-700 mb-1"
              >Họ</label
            >
            <input
              id="registerHo"
              v-model="registerHo"
              type="text"
              required
              placeholder="Nguyễn"
              class="input"
            />
          </div>

          <div>
            <label
              for="registerTenLot"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Tên lót</label
            >
            <input
              id="registerTenLot"
              v-model="registerTenLot"
              type="text"
              placeholder="Văn"
              class="input"
            />
          </div>

          <div>
            <label for="registerTen" class="block text-sm font-medium text-gray-700 mb-1"
              >Tên</label
            >
            <input
              id="registerTen"
              v-model="registerTen"
              type="text"
              required
              placeholder="An"
              class="input"
            />
          </div>

          <div>
            <label
              for="registerPhone"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Số điện thoại</label
            >
            <input
              id="registerPhone"
              v-model="registerPhone"
              type="tel"
              required
              placeholder="0123 456 789"
              class="input"
            />
          </div>

          <div class="col-span-2">
            <label
              for="registerEmail"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Email</label
            >
            <input
              id="registerEmail"
              v-model="registerEmail"
              type="email"
              required
              placeholder="Nhập email"
              class="input"
            />
          </div>

          <div>
            <label
              for="registerPassword"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Mật khẩu</label
            >
            <input
              id="registerPassword"
              v-model="registerPassword"
              type="password"
              required
              placeholder="Tạo mật khẩu"
              class="input"
            />
          </div>

          <div>
            <label
              for="registerConfirm"
              class="block text-sm font-medium text-gray-700 mb-1"
              >Xác nhận mật khẩu</label
            >
            <input
              id="registerConfirm"
              v-model="registerConfirm"
              type="password"
              required
              placeholder="Nhập lại mật khẩu"
              class="input"
            />
          </div>
        </div>
        <button type="submit" :disabled="loading" class="btn-primary mt-2">
          <span v-if="loading">Đang xử lý...</span>
          <span v-else>Đăng ký</span>
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
import { ref, defineExpose } from "vue";
import axios from "axios";
import Cookies from "js-cookie";
import { useRouter } from "vue-router";
import { useUserStore } from "../stores/userStore";

const show = ref(false);
const mode = ref("login");
const email = ref("");
const password = ref("");
const registerHo = ref("");
const registerTenLot = ref("");
const registerTen = ref("");
const registerEmail = ref("");
const registerPhone = ref("");
const registerPassword = ref("");
const registerConfirm = ref("");
const showPassword = ref(false);
const loading = ref(false);
const error = ref("");

// Toast state
const toast = ref({
  show: false,
  message: "",
  type: "success", // 'success' | 'error'
});

const router = useRouter();
const userStore = useUserStore();

// Hiển thị mật khẩu
const togglePassword = () => (showPassword.value = !showPassword.value);

// Mở / đóng popup
function open(defaultMode = "login") {
  mode.value = defaultMode;
  show.value = true;
}
function close() {
  show.value = false;
}

// Expose
defineExpose({ open, close });

// Toast
function showToast(message, type = "success") {
  toast.value = { show: true, message, type };
  setTimeout(() => (toast.value.show = false), 3000);
}

// Xử lý đăng nhập
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
      Cookies.set("jwt_token", token);
      userStore.setToken(token);
      await userStore.fetchUser();
      showToast("Đăng nhập thành công!", "success");
      close();
      router.push("/home");
    } else error.value = res.data.message || "Đăng nhập thất bại!";
  } catch (err) {
    error.value = "Lỗi kết nối đến máy chủ!";
  } finally {
    loading.value = false;
  }
};

// Xử lý đăng ký
const handleRegister = async () => {
  error.value = "";

  if (registerPassword.value !== registerConfirm.value) {
    error.value = "Mật khẩu xác nhận không khớp!";
    return;
  }

  const phoneRegex = /^[0-9]{10,11}$/;
  if (!phoneRegex.test(registerPhone.value)) {
    error.value = "Số điện thoại không hợp lệ!";
    return;
  }

  loading.value = true;
  try {
    const res = await axios.post("http://localhost:8082/api/users/create", {
      ho: registerHo.value,
      tenlot: registerTenLot.value,
      ten: registerTen.value,
      email: registerEmail.value,
      sdt: registerPhone.value,
      matkhau: registerPassword.value,
    });

    if (res.data.code === 200) {
      showToast("Tạo tài khoản thành công! Vui lòng đăng nhập.", "success");
      mode.value = "login";

      registerHo.value = "";
      registerTenLot.value = "";
      registerTen.value = "";
      registerEmail.value = "";
      registerPhone.value = "";
      registerPassword.value = "";
      registerConfirm.value = "";
    } else {
      error.value = res.data.message || "Đăng ký thất bại!";
    }
  } catch (err) {
    console.error(err);
    error.value = "Lỗi kết nối đến máy chủ!";
  } finally {
    loading.value = false;
  }
};
</script>

<style scoped>
@import "@/assets/styles/auth.css";
@import "@/assets/styles/toast.css";
</style>
