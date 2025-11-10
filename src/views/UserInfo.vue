<template>
  <div class="min-h-screen flex flex-col items-center bg-gray-50 py-10 px-4">
    <h2 class="text-2xl font-semibold text-gray-800 mb-8">THÔNG TIN TÀI KHOẢN</h2>

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

    <div v-if="loading" class="text-center text-gray-500 py-6">
      Đang tải thông tin người dùng...
    </div>

    <div v-else class="w-full max-w-3xl space-y-5">
      <!-- Thông tin mặc định -->
      <div class="space-y-3">
        <div class="flex items-center gap-4">
          <label class="w-40 text-sm text-gray-700 font-extrabold">Họ và tên</label>
          <p class="text-gray-900 flex-1">
            {{ `${user.ho} ${user.tenlot} ${user.ten}`.trim() }}
          </p>
        </div>

        <div class="flex items-center gap-4">
          <label class="w-40 text-sm font-extrabold text-gray-700">Email</label>
          <p class="text-gray-900 flex-1">{{ user.email }}</p>
        </div>

        <div class="flex items-center gap-4">
          <label class="w-40 text-sm font-extrabold text-gray-700">Mật khẩu</label>
          <div class="flex items-center gap-3 flex-1">
            <p class="text-gray-900 tracking-widest select-none">•••••</p>
            <a
              href="#"
              class="text-sky-600 hover:text-sky-700 text-sm font-medium underline"
              @click.prevent="openChangePassword"
              >Đổi mật khẩu</a
            >
          </div>
        </div>
      </div>

      <hr class="border-gray-300 my-4" />

      <!-- FORM -->
      <div class="space-y-4">
        <div class="form-row">
          <label>Họ</label>
          <div class="flex-1">
            <input
              v-model="user.ho"
              :disabled="!editing"
              :class="['input', errors.ho && 'error']"
            />
            <p v-if="errors.ho" class="error-msg">{{ errors.ho }}</p>
          </div>
        </div>

        <div class="form-row">
          <label>Tên lót</label>
          <div class="flex-1">
            <input
              v-model="user.tenlot"
              :disabled="!editing"
              :class="['input', errors.tenlot && 'error']"
            />
            <p v-if="errors.tenlot" class="error-msg">{{ errors.tenlot }}</p>
          </div>
        </div>

        <div class="form-row">
          <label>Tên</label>
          <div class="flex-1">
            <input
              v-model="user.ten"
              :disabled="!editing"
              :class="['input', errors.ten && 'error']"
            />
            <p v-if="errors.ten" class="error-msg">{{ errors.ten }}</p>
          </div>
        </div>

        <div class="form-row">
          <label>Số điện thoại</label>
          <div class="flex-1">
            <input
              v-model="user.sdt"
              :disabled="!editing"
              :class="['input', errors.sdt && 'error']"
            />
            <p v-if="errors.sdt" class="error-msg">{{ errors.sdt }}</p>
          </div>
        </div>

        <div class="form-row">
          <label>Địa chỉ</label>
          <div class="flex-1">
            <input
              v-model="user.diachi"
              placeholder="Chưa cập nhật"
              :disabled="!editing"
              :class="['input', errors.diachi && 'error']"
            />
            <p v-if="errors.diachi" class="error-msg">{{ errors.diachi }}</p>
          </div>
        </div>

        <div class="form-row">
          <label>Địa chỉ giao hàng</label>
          <div class="flex-1">
            <input
              v-model="user.diachigiaohang"
              placeholder="Chưa cập nhật"
              :disabled="!editing"
              :class="['input', errors.diachigiaohang && 'error']"
            />
            <p v-if="errors.diachigiaohang" class="error-msg">
              {{ errors.diachigiaohang }}
            </p>
          </div>
        </div>

        <div class="form-row">
          <label>Thành phố</label>
          <div class="flex-1">
            <select
              v-model="user.thanhPho.matp"
              :disabled="!editing"
              :class="['input', errors.matp && 'error']"
            >
              <option disabled value="default">-- Chưa chọn thành phố --</option>
              <option v-for="tp in cities" :key="tp.matp" :value="tp.matp">
                {{ tp.tentp }}
              </option>
            </select>
            <p v-if="errors.matp" class="error-msg">{{ errors.matp }}</p>
          </div>
        </div>
      </div>

      <!-- BTN -->
      <div class="pt-6 flex justify-end gap-3">
        <button v-if="!editing" @click="enableEdit" class="btn-action">Chỉnh sửa</button>
        <button v-else @click="saveChanges" class="btn-action">Lưu thay đổi</button>
        <button v-if="editing" @click="cancelEdit" class="btn-cancel">Hủy</button>
      </div>
    </div>
  </div>
  <ChangePasswordPopup ref="changePasswordPopup" />
</template>

<script setup>
import { ref, onMounted } from "vue";
import axios from "axios";
import Cookies from "js-cookie";
import ChangePasswordPopup from "@/components/ChangePasswordPopup.vue";

const changePasswordPopup = ref(null);
const user = ref(null);
const cities = ref([]);
const loading = ref(true);
const editing = ref(false);
const originalData = ref(null);
const errors = ref({});
const toast = ref({ show: false, message: "", type: "success" });

// Toast
const showToast = (message, type = "success") => {
  toast.value = { show: true, message, type };
  setTimeout(() => (toast.value.show = false), 3000);
};

// Edit
const enableEdit = () => {
  editing.value = true;
  originalData.value = JSON.parse(JSON.stringify(user.value));
  errors.value = {};
};

const cancelEdit = () => {
  if (originalData.value) user.value = JSON.parse(JSON.stringify(originalData.value));
  editing.value = false;
  errors.value = {};
};

// Validate
const validateUserData = () => {
  errors.value = {};
  const phoneRegex = /^(09|08|07|05|03)[0-9]{8,9}$/;

  if (!user.value.ho?.trim()) errors.value.ho = "Vui lòng nhập họ.";
  if (!user.value.tenlot?.trim()) errors.value.tenlot = "Vui lòng nhập tên lót.";
  if (!user.value.ten?.trim()) errors.value.ten = "Vui lòng nhập tên.";
  if (!user.value.sdt?.trim()) {
    errors.value.sdt = "Vui lòng nhập số điện thoại.";
  } else if (!/^[0-9]{10,11}$/.test(user.value.sdt)) {
    errors.value.sdt = "Số điện thoại phải gồm 10–11 chữ số.";
  } else if (!phoneRegex.test(user.value.sdt)) {
    errors.value.sdt = "Số điện thoại phải bắt đầu bằng 09, 08, 07, 05 hoặc 03.";
  }
  if (!user.value.diachi?.trim()) errors.value.diachi = "Vui lòng nhập địa chỉ.";
  if (!user.value.diachigiaohang?.trim())
    errors.value.diachigiaohang = "Vui lòng nhập địa chỉ giao hàng.";
  if (!user.value.thanhPho?.matp || user.value.thanhPho.matp === "default")
    errors.value.matp = "Vui lòng chọn thành phố.";

  return Object.keys(errors.value).length === 0;
};

// Save
const saveChanges = async () => {
  if (!validateUserData()) return;

  try {
    const token = Cookies.get("jwt_token");
    if (!token) {
      showToast("Phiên đăng nhập đã hết hạn, vui lòng đăng nhập lại.", "error");
      return;
    }

    const payload = {
      ho: user.value.ho.trim(),
      tenlot: user.value.tenlot.trim(),
      ten: user.value.ten.trim(),
      diachi: user.value.diachi.trim(),
      diachigiaohang: user.value.diachigiaohang.trim(),
      sdt: user.value.sdt.trim(),
      matp: user.value.thanhPho.matp,
    };

    const res = await axios.put("http://localhost:8082/api/users/update-info", payload, {
      headers: { Authorization: `Bearer ${token}` },
    });

    if (res.data?.code === 200) {
      user.value = res.data.result;
      editing.value = false;
      errors.value = {};
      showToast("Cập nhật thông tin thành công!", "success");
    } else {
      showToast(res.data?.message || "Lưu thất bại, vui lòng thử lại!", "error");
    }
  } catch (err) {
    console.error(err);
    showToast("Lưu thất bại, vui lòng thử lại!", "error");
  }
};

// Mounted
onMounted(async () => {
  try {
    const token = Cookies.get("jwt_token");
    if (!token) {
      showToast("Bạn chưa đăng nhập!", "error");
      return;
    }

    const [meRes, citiesRes] = await Promise.all([
      axios.get("http://localhost:8082/api/auth/me", {
        headers: { Authorization: `Bearer ${token}` },
      }),
      axios.get("http://localhost:8082/api/cities/find-all"),
    ]);

    if (meRes.data?.result) {
      const u = meRes.data.result;
      user.value = {
        ...u,
        diachi: u.diachi ?? "",
        diachigiaohang: u.diachigiaohang ?? "",
        thanhPho: u.thanhPho ?? { matp: "default" },
      };
    }

    if (citiesRes.data?.result) cities.value = citiesRes.data.result;
  } catch (err) {
    console.error(err);
    showToast("Không thể tải thông tin người dùng!", "error");
  } finally {
    loading.value = false;
  }
});

// Pass popup
function openChangePassword() {
  changePasswordPopup.value?.open();
}
</script>

<style scoped>
@import "@/assets/styles/userinfo.css";
@import "@/assets/styles/toast.css";
</style>
