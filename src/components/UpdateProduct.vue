<template>
  <div
    v-if="visible"
    class="fixed inset-0 z-50 flex items-center justify-center bg-black/50 backdrop-blur-sm"
  >
    <div class="relative z-10 w-full max-w-3xl rounded-lg bg-white p-8 shadow-lg">
      <div class="relative mb-6 border-b pb-3">
        <h2 class="text-2xl font-semibold text-gray-800 text-center">
          CẬP NHẬT SẢN PHẨM
        </h2>
        <button
          class="absolute right-0 top-1/2 -translate-y-1/2 text-gray-500 hover:text-gray-700 text-xl"
          @click="close"
          aria-label="Đóng"
        >
          ✕
        </button>
      </div>

      <!-- Form -->
      <form class="space-y-5">
        <div class="form-row">
          <label>Thành phố</label>
          <div class="flex-1">
            <select v-model="form.thanhPho.matp" class="input">
              <option disabled value="">-- Chọn thành phố --</option>
              <option v-for="c in cities" :key="c.matp" :value="String(c.matp)">
                {{ c.tentp }}
              </option>
            </select>
          </div>
        </div>

        <div class="form-row">
          <label>Danh mục</label>
          <div class="flex-1">
            <select v-model="form.danhMuc.madm" class="input">
              <option disabled value="">-- Chọn danh mục --</option>
              <option v-for="cat in categories" :key="cat.madm" :value="String(cat.madm)">
                {{ cat.tendm }}
              </option>
            </select>
          </div>
        </div>

        <div class="form-row">
          <label>Tên sản phẩm</label>
          <input v-model="form.tensp" class="input flex-1" />
        </div>

        <div class="form-row">
          <label>Tình trạng</label>
          <input v-model="form.tinhtrangsp" class="input flex-1" />
        </div>

        <div class="form-row flex-col items-start">
          <label>Ảnh sản phẩm (tối đa 3 ảnh)</label>
          <div class="flex gap-4 flex-wrap mt-2">
            <div
              v-for="(img, index) in form.hinhAnh"
              :key="index"
              class="w-32 h-32 border rounded-md flex items-center justify-center bg-gray-100 relative"
            >
              <img
                v-if="img.preview || img.tenanh"
                :src="img.preview || getImageUrl(img.tenanh)"
                class="object-cover w-full h-full rounded-md"
              />
              <button
                type="button"
                @click.stop="removeImage(index)"
                class="absolute top-1 right-1 text-red-500 bg-white rounded-full w-5 h-5 flex items-center justify-center text-sm shadow-sm"
              >
                ✕
              </button>
            </div>

            <label
              v-if="form.hinhAnh.length < 3"
              class="w-32 h-32 border border-dashed rounded-md flex items-center justify-center cursor-pointer text-gray-400"
            >
              +
              <input
                ref="imageInput"
                type="file"
                class="hidden"
                accept="image/png, image/jpeg, image/jpg"
                @change="onAddImage"
              />
            </label>
          </div>
        </div>
      </form>

      <!-- Footer -->
      <div class="pt-6 flex justify-end gap-3 border-t mt-6">
        <button
          class="px-4 py-2 rounded-lg text-gray-700 hover:bg-gray-100"
          @click="close"
        >
          Hủy
        </button>
        <button type="button" class="btn-primary max-w-[130px]" @click="save">Lưu</button>
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
import { reactive, ref, watch, onMounted } from "vue";
import axios from "axios";
import Cookies from "js-cookie";

const props = defineProps({
  visible: Boolean,
  product: Object,
});

const emit = defineEmits(["close", "updated"]);

const imageInput = ref(null);
const categories = ref([]);
const cities = ref([]);

const form = reactive({
  masp: "",
  tensp: "",
  tinhtrangsp: "",
  thanhPho: { matp: "" },
  danhMuc: { madm: "" },
  hinhAnh: [],
});

// Toast
const toast = reactive({ show: false, message: "", type: "success" });
function showToast(msg, type = "success") {
  toast.message = msg;
  toast.type = type;
  toast.show = true;
  setTimeout(() => (toast.show = false), 2500);
}

const getImageUrl = (tenanh) => `http://localhost:8082/api/imgs/${tenanh}`;

onMounted(async () => {
  try {
    const [catRes, cityRes] = await Promise.all([
      axios.get("http://localhost:8082/api/cates/find-all"),
      axios.get("http://localhost:8082/api/cities/find-all"),
    ]);
    categories.value = catRes.data?.result || [];
    cities.value = cityRes.data?.result || [];
  } catch (err) {
    console.error("Không thể tải danh mục/thành phố:", err);
  }
});

// Load product khi mở modal
watch(
  () => props.product,
  (p) => {
    if (!p) return;
    Object.assign(form, {
      masp: p.masp ?? "",
      tensp: p.tensp ?? "",
      tinhtrangsp: p.tinhtrangsp ?? "",
      thanhPho: { matp: p.thanhPho?.matp ? String(p.thanhPho.matp) : "" },
      danhMuc: { madm: p.danhMuc?.madm ? String(p.danhMuc.madm) : "" },
      hinhAnh: (p.hinhAnh || []).map((img) => ({
        tenanh: img.tenanh,
        file: null,
        preview: null,
      })),
    });
  },
  { immediate: true }
);

function onAddImage(e) {
  const file = e.target.files?.[0];
  if (!file) return;
  if (form.hinhAnh.length >= 3) return showToast("Tối đa 3 ảnh", "error");

  const safeName = file.name.replace(/\s+/g, "-").toLowerCase();
  const renamedFile = new File([file], safeName, { type: file.type });
  const reader = new FileReader();
  reader.onload = () => {
    form.hinhAnh.push({ tenanh: safeName, file: renamedFile, preview: reader.result });
  };
  reader.readAsDataURL(renamedFile);
  e.target.value = "";
}

function removeImage(i) {
  form.hinhAnh.splice(i, 1);
}

function close() {
  emit("close");
}

async function save() {
  const token = Cookies.get("jwt_token");
  if (!token) return showToast("Vui lòng đăng nhập lại!", "error");

  try {
    const body = {
      masp: form.masp,
      madm: form.danhMuc.madm,
      matp: form.thanhPho.matp,
      tensp: form.tensp,
      tinhtrangsp: form.tinhtrangsp,
    };

    const res = await axios.put("http://localhost:8082/api/products/update", body, {
      headers: { Authorization: `Bearer ${token}` },
    });

    if (res.data?.code !== 200) throw new Error(res.data?.message || "Cập nhật thất bại");

    // Cập nhật ảnh nếu có
    const files = form.hinhAnh.filter((img) => img.file);
    if (files.length) {
      const formData = new FormData();
      formData.append("masp", form.masp);
      files.forEach((img) => formData.append("files", img.file));
      await axios.put("http://localhost:8082/api/images/update", formData, {
        headers: { Authorization: `Bearer ${token}` },
      });
    }

    showToast("Cập nhật thành công!");
    emit("updated"); // 👈 báo cho cha reload
  } catch (err) {
    console.error(err);
    showToast("Có lỗi khi cập nhật!", "error");
  }
}
</script>

<style scoped>
@import "@/assets/styles/auth.css";
@import "@/assets/styles/toast.css";
.form-row {
  @apply flex flex-col md:flex-row md:items-center gap-3;
}
.form-row label {
  @apply w-40 font-medium text-gray-700;
}
</style>
