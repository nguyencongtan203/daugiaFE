<template>
  <div class="auction-room" v-if="user">
    <h2>🏷️ Phòng đấu giá: {{ roomId }}</h2>

    <div class="info">
      <p><b>👤 Người dùng:</b> {{ user.email }}</p>
      <p><b>💰 Giá cao nhất hiện tại:</b> {{ highestPrice.toLocaleString() }}đ</p>
    </div>

    <div class="input">
      <label>Số lần trả giá:</label>
      <input type="number" v-model.number="bidStep" min="1" />

      <button @click="sendBid" :disabled="!stompConnected || isWaiting">
        <span v-if="!isWaiting">Trả giá</span>
        <span v-else>⏳ Chờ {{ countdown }}s</span>
      </button>
    </div>

    <p v-if="errorMsg" class="error-box">{{ errorMsg }}</p>

    <hr />
    <h3>📜 Lịch sử trả giá</h3>
    <ul>
      <li v-for="(bid, i) in bids" :key="i">
        <b>{{ bid.taiKhoanNguoiRaGia?.matk }}</b>
        ➜ {{ bid.sotien.toLocaleString() }}đ ({{ bid.solan }} lần)
      </li>
    </ul>
  </div>

  <div v-else class="loading">
    <p>🔄 Đang kiểm tra đăng nhập...</p>
  </div>
</template>

<script setup>
import SockJS from "sockjs-client";
import Stomp from "stompjs";
import { ref, onMounted, onUnmounted } from "vue";
import { useRouter } from "vue-router";
import { useUserStore } from "../stores/userStore";
import { storeToRefs } from "pinia";

const router = useRouter();
const userStore = useUserStore();
const { user, token } = storeToRefs(userStore);

const socketUrl = "http://localhost:8082/api/ws-auction";
const roomId = ref("1");
const bidStep = ref(1);
const highestPrice = ref(0);
const bids = ref([]);
const errorMsg = ref("");
const stompConnected = ref(false);

// ⏳ Biến điều khiển đếm ngược
const isWaiting = ref(false);
const countdown = ref(20);
let countdownTimer = null;

let stompClient = null;

onMounted(async () => {
  // 🧩 Kiểm tra đăng nhập
  if (!token.value) {
    router.push("/login");
    return;
  }

  if (!user.value) {
    await userStore.fetchUser();
  }

  if (!user.value) {
    router.push("/login");
    return;
  }

  // 🔌 Kết nối WebSocket
  connectWebSocket();
});

function connectWebSocket() {
  const socket = new SockJS(socketUrl);
  stompClient = Stomp.over(socket);
  stompClient.debug = null; // Tắt log debug

  stompClient.connect(
    {},
    () => {
      console.log("✅ Đã kết nối WebSocket");
      stompConnected.value = true;

      stompClient.subscribe(`/topic/auction/${roomId.value}`, (message) => {
        try {
          const data = JSON.parse(message.body);
          console.log("📩 Nhận:", data);

          if (data.error) {
            errorMsg.value = data.message || "Đã xảy ra lỗi!";
            setTimeout(() => (errorMsg.value = ""), 4000);
            return;
          }

          // Nếu chính user này là người vừa bid thành công
          if (data.taiKhoanNguoiRaGia?.matk === user.value.matk) {
            startCountdown(20); // Khóa nút 20 giây
          }

          bids.value.unshift(data);
          highestPrice.value = data.sotien || highestPrice.value;
        } catch (e) {
          console.warn("Không thể parse message:", e);
        }
      });
    },
    (err) => {
      console.warn("⚠️ Mất kết nối WebSocket:", err);
      stompConnected.value = false;
      // Thử reconnect sau 3s
      setTimeout(connectWebSocket, 3000);
    }
  );
}

function sendBid() {
  if (!stompClient || !stompClient.connected) {
    errorMsg.value = "⚠️ Chưa kết nối WebSocket!";
    setTimeout(() => (errorMsg.value = ""), 3000);
    return;
  }

  if (!user.value) {
    errorMsg.value = "❌ Bạn chưa đăng nhập!";
    setTimeout(() => (errorMsg.value = ""), 3000);
    return;
  }

  const payload = {
    phienDauGia: { maphiendg: roomId.value },
    taiKhoanNguoiRaGia: { matk: user.value.matk },
    solan: bidStep.value,
  };

  console.log("📤 Gửi bid:", payload);
  stompClient.send("/app/bid", {}, JSON.stringify(payload));
}

// 🕒 Hàm bắt đầu đếm ngược 20s
function startCountdown(seconds) {
  if (countdownTimer) clearInterval(countdownTimer);

  isWaiting.value = true;
  countdown.value = seconds;

  countdownTimer = setInterval(() => {
    countdown.value--;
    if (countdown.value <= 0) {
      isWaiting.value = false;
      clearInterval(countdownTimer);
    }
  }, 1000);
}

onUnmounted(() => {
  if (stompClient) {
    stompClient.disconnect(() => console.log("🔌 Đã ngắt WebSocket"));
  }
  if (countdownTimer) clearInterval(countdownTimer);
});
</script>

<style scoped>
.auction-room {
  background: white;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.info {
  background: #f9f9f9;
  padding: 10px 15px;
  border-radius: 8px;
  margin-bottom: 15px;
}

.input {
  display: flex;
  align-items: center;
  gap: 10px;
  margin: 10px 0;
}

input {
  width: 80px;
  padding: 5px;
  border-radius: 6px;
  border: 1px solid #ccc;
}

button {
  background: #42b983;
  color: white;
  border: none;
  padding: 6px 14px;
  border-radius: 6px;
  cursor: pointer;
  font-weight: bold;
  transition: background 0.3s;
}

button:hover {
  background: #369b73;
}

button:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.loading {
  text-align: center;
  padding: 40px;
  font-size: 18px;
  color: #666;
}

.error-box {
  color: #e74c3c;
  background: #fdecea;
  padding: 10px;
  border-radius: 6px;
  margin: 10px 0;
  font-weight: bold;
}
</style>
