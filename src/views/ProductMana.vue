<template>
  <div class="min-h-screen product-mana p-4 bg-gray-50 py-10">
    <!-- Tabs -->
    <div class="menu flex gap-4 mb-4 border-b pb-2">
      <button
        :class="{ active: activeTab === 'products' }"
        @click="activeTab = 'products'"
        class="px-4 py-2 rounded"
      >
        Sản phẩm đã đăng ký
      </button>
      <button
        :class="{ active: activeTab === 'auctions' }"
        @click="activeTab = 'auctions'"
        class="px-4 py-2 rounded"
      >
        Phiên đấu giá
      </button>
    </div>

    <!-- Danh sách sản phẩm -->
    <div v-if="activeTab === 'products'">
      <table class="w-full border-collapse">
        <thead>
          <tr class="bg-gray-200">
            <th class="border p-2">Hình ảnh</th>
            <th class="border p-2">Mã sản phẩm</th>
            <th class="border p-2">Tên sản phẩm</th>
            <th class="border p-2">Trạng thái</th>
            <th class="border p-2">Hành động</th>
          </tr>
        </thead>
        <tbody>
          <template v-for="p in filteredProducts" :key="p.masp">
            <tr>
              <td class="border p-2 text-center align-middle">
                <div class="flex justify-center items-center h-36 w-full">
                  <template v-if="p.hinhAnh?.length">
                    <img
                      :src="getImageUrl(p.hinhAnh[0].tenanh)"
                      class="product-main-img object-cover rounded-lg max-h-36 max-w-36"
                    />
                  </template>
                  <template v-else>
                    <span class="text-gray-400 text-sm">Không có ảnh</span>
                  </template>
                </div>
              </td>
              <td class="border p-2 text-center">{{ p.masp }}</td>
              <td class="border p-2 text-center">{{ p.tensp }}</td>
              <td class="border p-2 text-center">{{ p.trangthai }}</td>
              <td class="border p-2 text-center">
                <button
                  class="px-3 py-1 bg-gray-200 rounded text-sm hover:bg-gray-300"
                  @click="toggleDetails(p.masp)"
                >
                  Chi tiết
                </button>

                <template v-if="p.trangthai !== 'Đã duyệt'">
                  <button
                    class="ml-2 px-3 py-1 bg-blue-500 text-white rounded text-sm hover:bg-blue-600"
                    @click="openEdit(p)"
                  >
                    Chỉnh sửa
                  </button>
                </template>

                <template v-else>
                  <button
                    class="ml-2 px-3 py-1 bg-green-500 text-white rounded text-sm hover:bg-green-600"
                    @click="openAuction(p)"
                  >
                    Tạo phiên
                  </button>
                </template>
              </td>
            </tr>

            <tr>
              <td colspan="5" class="p-0">
                <div class="dropdown-wrapper" :class="{ open: openedDetails === p.masp }">
                  <table class="w-full border-collapse dropdown-table">
                    <thead>
                      <tr class="bg-gray-100">
                        <th class="border p-1">Ảnh khác</th>
                        <th class="border p-1">Thành phố</th>
                        <th class="border p-1">Tình trạng</th>
                        <th class="border p-1">Danh mục</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr>
                        <td class="border p-2 text-center">
                          <div
                            v-if="p.hinhAnh?.length > 1"
                            class="flex justify-center gap-2 flex-wrap"
                          >
                            <img
                              v-for="(img, idx) in p.hinhAnh.slice(1)"
                              :key="idx"
                              :src="getImageUrl(img.tenanh)"
                              class="dropdown-img"
                            />
                          </div>
                          <div v-else class="text-gray-400 text-sm">
                            Không có ảnh thêm
                          </div>
                        </td>
                        <td class="border p-2 text-center">
                          {{ p.thanhPho?.tentp || "N/A" }}
                        </td>
                        <td class="border p-2 text-center">{{ p.tinhtrangsp }}</td>
                        <td class="border p-2 text-center">
                          {{ p.danhMuc?.tendm || "N/A" }}
                        </td>
                      </tr>
                    </tbody>
                  </table>
                </div>
              </td>
            </tr>
          </template>
        </tbody>
      </table>

      <div v-if="!filteredProducts.length" class="mt-4 text-gray-500 text-center">
        Không có sản phẩm nào.
      </div>
    </div>

    <!-- Danh sách phiên đấu giá -->
    <div v-else>
      <table class="w-full border-collapse">
        <thead>
          <tr class="bg-gray-200">
            <th class="border p-2">Mã phiên</th>
            <th class="border p-2">Giá khởi điểm</th>
            <th class="border p-2">Giá trần</th>
            <th class="border p-2">Bước giá</th>
            <th class="border p-2">Tiền cọc</th>
            <th class="border p-2">Giá cao nhất</th>
            <th class="border p-2">Trạng thái</th>
            <th class="border p-2">Kết quả</th>
            <th class="border p-2">Người tham gia</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="a in auctions" :key="a.maphiendg">
            <td class="border p-2 text-center">{{ a.maphiendg }}</td>
            <td class="border p-2 text-center">{{ a.giakhoidiem }}</td>
            <td class="border p-2 text-center">{{ a.giatran }}</td>
            <td class="border p-2 text-center">{{ a.buocgia }}</td>
            <td class="border p-2 text-center">{{ a.tiencoc }}</td>
            <td class="border p-2 text-center">{{ a.giacaonhatdatduoc }}</td>
            <td class="border p-2 text-center">{{ a.trangthai }}</td>
            <td class="border p-2 text-center">{{ a.ketquaphien }}</td>
            <td class="border p-2 text-center">{{ a.slnguoithamgia }}</td>
          </tr>
        </tbody>
      </table>

      <div v-if="!auctions.length" class="mt-4 text-gray-500 text-center">
        Không có phiên đấu giá nào.
      </div>
    </div>

    <!-- Popup -->
    <UpdateProduct
      :visible="showUpdatePopup"
      :product="editedProduct"
      @close="showUpdatePopup = false"
      @updated="handleUpdated"
    />

    <CreateAuction
      :visible="showAuctionPopup"
      :product="auctionProduct"
      @close="showAuctionPopup = false"
      @created="handleAuctionCreated"
    />
  </div>
</template>

<script setup>
import { ref, onMounted, computed } from "vue";
import axios from "axios";
import Cookies from "js-cookie";
import UpdateProduct from "@/components/UpdateProduct.vue";
import CreateAuction from "@/components/CreateAuction.vue";

const activeTab = ref("products");
const products = ref([]);
const auctions = ref([]);
const openedDetails = ref(null);

const showUpdatePopup = ref(false);
const showAuctionPopup = ref(false);

const editedProduct = ref(null);
const auctionProduct = ref(null);

const getImageUrl = (tenanh) => `http://localhost:8082/api/imgs/${tenanh}`;

const toggleDetails = (masp) => {
  openedDetails.value = openedDetails.value === masp ? null : masp;
};

const openEdit = (product) => {
  editedProduct.value = JSON.parse(JSON.stringify(product));
  showUpdatePopup.value = true;
};

const openAuction = (product) => {
  auctionProduct.value = JSON.parse(JSON.stringify(product));
  showAuctionPopup.value = true;
};

const handleUpdated = async () => {
  await fetchProducts();
  showUpdatePopup.value = false;
};

const handleAuctionCreated = async () => {
  await fetchAuctions();
  showAuctionPopup.value = false;
};

// Lọc sản phẩm theo trạng thái
const filteredProducts = computed(() =>
  products.value.filter((p) => ["Đã duyệt", "Chờ duyệt", "Đã hủy"].includes(p.trangthai))
);

// API: Lấy danh sách sản phẩm
const fetchProducts = async () => {
  try {
    const token = Cookies.get("jwt_token");
    if (!token) return;
    const res = await axios.get("http://localhost:8082/api/products/find-by-user", {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (res.data.code === 200) products.value = res.data.result || [];
  } catch (err) {
    console.error("Lỗi khi tải sản phẩm:", err);
  }
};

// API: Lấy danh sách phiên đấu giá
const fetchAuctions = async () => {
  try {
    const token = Cookies.get("jwt_token");
    if (!token) return;
    const res = await axios.get("http://localhost:8082/api/auctions/find-by-user", {
      headers: { Authorization: `Bearer ${token}` },
    });
    if (res.data.code === 200) auctions.value = res.data.result || [];
  } catch (err) {
    console.error("Lỗi khi tải phiên đấu giá:", err);
  }
};

onMounted(() => {
  fetchProducts();
  fetchAuctions();
});
</script>

<style scoped>
@import "@/assets/styles/productmana.css";
</style>
