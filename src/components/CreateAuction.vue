<template>
  <div
    v-if="visible"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm"
  >
    <div class="relative z-10 w-full max-w-6xl rounded-lg bg-white p-8 shadow-lg">
      <!-- Header -->
      <div class="flex justify-center mb-6 border-b pb-3 relative">
        <h2 class="text-2xl font-semibold text-gray-800 text-center">
          Tạo phiên đấu giá
        </h2>
        <button
          class="absolute right-0 top-0 text-gray-500 hover:text-gray-700 text-xl"
          @click="close"
          aria-label="Đóng"
        >
          ✕
        </button>
      </div>

      <!-- Form -->
      <form class="grid grid-cols-1 md:grid-cols-2 gap-5">
        <!-- Mã sản phẩm -->
        <div class="form-row">
          <label>Mã sản phẩm</label>
          <p class="font-semibold text-gray-700 w-full">{{ form.masp }}</p>
        </div>

        <!-- Trường số -->
        <template v-for="f in fields" :key="f.key">
          <div class="form-row">
            <label>{{ f.label }}</label>
            <div class="flex flex-col w-full">
              <input
                v-model="form[f.key]"
                @input="validateNumber(f.key)"
                type="text"
                inputmode="numeric"
                :class="['input', errors[f.key] ? 'border-red-500' : '']"
                :placeholder="`Nhập ${f.label.toLowerCase()}`"
              />
              <small v-if="errors[f.key]" class="text-red-500 text-sm mt-1">{{
                errors[f.key]
              }}</small>
            </div>
          </div>
        </template>

        <!-- Trường thời gian -->
        <template v-for="t in timeFields" :key="t.key">
          <div class="form-row">
            <label>{{ t.label }}</label>
            <div class="flex flex-col w-full">
              <input
                v-model="form[t.key]"
                type="datetime-local"
                @change="validateTimes"
                :class="['input', errors[t.key] ? 'border-red-500' : '']"
              />
              <small v-if="errors[t.key]" class="text-red-500 text-sm mt-1">{{
                errors[t.key]
              }}</small>
            </div>
          </div>
        </template>
      </form>

      <!-- Actions -->
      <div class="pt-6 flex justify-end gap-3 border-t mt-6">
        <button
          class="px-4 py-2 rounded-lg text-gray-700 hover:bg-gray-100"
          @click="close"
        >
          Hủy
        </button>
        <button type="button" class="btn-primary max-w-[180px]" @click="save">
          Tạo phiên
        </button>
      </div>
    </div>
  </div>
  <!-- Toast -->
  <transition name="slide-fade">
    <div
      v-if="toast.show"
      :class="[
        'fixed top-5 right-5 min-w-[250px] px-4 py-3 rounded-lg shadow-md text-gray-800 bg-white border-l-4 z-[9999]',
        toast.type === 'success' ? 'border-green-500' : 'border-red-500',
      ]"
    >
      {{ toast.message }}
    </div>
  </transition>
</template>

<script setup>
import { reactive, watch } from "vue";
import axios from "axios";
import Cookies from "js-cookie";

const props = defineProps({
  visible: Boolean,
  product: Object,
});
const emit = defineEmits(["close", "created"]);

const form = reactive({
  masp: "",
  giakhoidiem: "",
  giatran: "",
  buocgia: "",
  tiencoc: "",
  thoigianbd: "",
  thoigiankt: "",
  thoigianbddk: "",
  thoigianktdk: "",
});

const errors = reactive({});

const fields = [
  { key: "giakhoidiem", label: "Giá khởi điểm" },
  { key: "giatran", label: "Giá trần" },
  { key: "buocgia", label: "Bước giá" },
  { key: "tiencoc", label: "Tiền cọc" },
];

const timeFields = [
  { key: "thoigianbddk", label: "Thời gian bắt đầu đăng ký" },
  { key: "thoigianktdk", label: "Thời gian kết thúc đăng ký" },
  { key: "thoigianbd", label: "Thời gian bắt đầu phiên" },
  { key: "thoigiankt", label: "Thời gian kết thúc phiên" },
];

// Reset khi mở popup
watch(
  () => props.product,
  (p) => {
    if (!p) return;
    Object.assign(form, {
      masp: p.masp,
      giakhoidiem: "",
      giatran: "",
      buocgia: "",
      tiencoc: "",
      thoigianbd: "",
      thoigiankt: "",
      thoigianbddk: "",
      thoigianktdk: "",
    });
    Object.keys(errors).forEach((k) => (errors[k] = ""));
  },
  { immediate: true }
);

// Toast
const toast = reactive({ show: false, message: "", type: "success" });
const showToast = (msg, type = "success") => {
  toast.message = msg;
  toast.type = type;
  toast.show = true;
  setTimeout(() => (toast.show = false), 3000);
};

const close = () => emit("close");

// ✅ Validate số
const validateNumber = (field) => {
  const raw = form[field] || "";
  const val = raw.replace(/[^0-9]/g, "");
  if (raw !== val) form[field] = val;

  if (!val) errors[field] = "Vui lòng nhập số tiền.";
  else if (Number(val) < 0) errors[field] = "Giá trị phải là số không âm.";
  else errors[field] = "";
};

// ✅ Validate thời gian
const validateTimes = () => {
  const now = new Date();
  const startAuction = new Date(form.thoigianbd);
  const endAuction = new Date(form.thoigiankt);
  const startReg = new Date(form.thoigianbddk);
  const endReg = new Date(form.thoigianktdk);

  timeFields.forEach(({ key }) => (errors[key] = ""));

  if (!form.thoigianbd) errors.thoigianbd = "Vui lòng chọn thời gian bắt đầu phiên.";
  if (!form.thoigiankt) errors.thoigiankt = "Vui lòng chọn thời gian kết thúc phiên.";
  if (!form.thoigianbddk)
    errors.thoigianbddk = "Vui lòng chọn thời gian bắt đầu đăng ký.";
  if (!form.thoigianktdk)
    errors.thoigianktdk = "Vui lòng chọn thời gian kết thúc đăng ký.";

  if (form.thoigianbd && startAuction < now)
    errors.thoigianbd = "Thời gian bắt đầu phiên không được là ngày trong quá khứ.";
  if (form.thoigiankt && endAuction <= startAuction)
    errors.thoigiankt = "Thời gian kết thúc phiên phải sau thời gian bắt đầu.";
  if (form.thoigianbddk && startReg < now)
    errors.thoigianbddk = "Thời gian bắt đầu đăng ký không được là ngày trong quá khứ.";
  if (form.thoigianbddk && form.thoigianbd && startReg >= startAuction)
    errors.thoigianbddk = "Thời gian bắt đầu đăng ký phải trước thời gian bắt đầu phiên.";
  if (form.thoigianktdk && endReg <= startReg)
    errors.thoigianktdk =
      "Thời gian kết thúc đăng ký phải sau thời gian bắt đầu đăng ký.";
  if (form.thoigianktdk && endReg >= startAuction)
    errors.thoigianktdk =
      "Thời gian kết thúc đăng ký phải trước thời gian bắt đầu phiên.";
};

watch(
  () => [form.thoigianbd, form.thoigiankt, form.thoigianbddk, form.thoigianktdk],
  () => validateTimes()
);

// ✅ Format datetime cho backend (có giây)
const formatDateTime = (val) => {
  if (!val) return "";
  const localDate = new Date(val);
  const utcDate = new Date(localDate.getTime() - localDate.getTimezoneOffset() * 60000);
  return utcDate.toISOString().slice(0, 19).replace("T", " ");
};

// ✅ Lưu
const save = async () => {
  const token = Cookies.get("jwt_token");
  if (!token) return showToast("Vui lòng đăng nhập lại!", "error");

  fields.forEach(({ key }) => validateNumber(key));
  validateTimes();

  if (Object.values(errors).some((e) => e))
    return showToast("Vui lòng kiểm tra lại thông tin nhập!", "error");

  try {
    const payload = {
      masp: form.masp,
      thoigianbd: formatDateTime(form.thoigianbd),
      thoigiankt: formatDateTime(form.thoigiankt),
      thoigianbddk: formatDateTime(form.thoigianbddk),
      thoigianktdk: formatDateTime(form.thoigianktdk),
      giakhoidiem: form.giakhoidiem,
      giatran: form.giatran,
      buocgia: form.buocgia,
      tiencoc: form.tiencoc,
    };

    const res = await axios.post("http://localhost:8082/api/auctions/create", payload, {
      headers: { Authorization: `Bearer ${token}` },
    });

    if (res.data.code !== 200) throw new Error(res.data.message || "Tạo phiên thất bại");

    showToast("Tạo phiên đấu giá thành công!");
    emit("created");
  } catch (err) {
    console.error(err);
    showToast("Lỗi khi tạo phiên đấu giá!", "error");
  }
};
</script>

<style scoped>
@import "@/assets/styles/auth.css";

.form-row {
  @apply flex flex-col md:flex-row md:items-start gap-3;
}
.form-row label {
  @apply w-52 font-medium text-gray-700;
}
.input {
  @apply border border-gray-300 rounded-md px-3 py-2 w-full focus:ring focus:ring-blue-200 outline-none transition;
}
.input.border-red-500 {
  @apply border-red-500 focus:ring-red-200;
}
.btn-primary {
  @apply px-4 py-2 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition;
}
</style>
