<template>
  <div class="mine-page">
    <!-- 用户信息卡片 -->
    <div class="user-card">
      <div class="avatar">
        <van-image
          v-if="userInfo.avatar"
          round
          width="60"
          height="60"
          :src="userInfo.avatar"
        />
        <div v-else class="avatar-placeholder">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" width="28" height="28">
            <path d="M20 21v-2a4 4 0 0 0-4-4H8a4 4 0 0 0-4 4v2"/>
            <circle cx="12" cy="7" r="4"/>
          </svg>
        </div>
      </div>
      <div class="user-info">
        <div class="nickname">{{ userInfo.nickname || '未登录' }}</div>
        <div class="user-id" v-if="userInfo.id">ID: {{ userInfo.id }}</div>
      </div>
    </div>

    <!-- 功能列表 -->
    <van-cell-group inset class="menu-group">
      <van-cell title="收货地址" is-link>
        <template #icon>
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18" style="margin-right:8px">
            <path d="M21 10c0 7-9 13-9 13s-9-6-9-13a9 9 0 0 1 18 0z"/>
            <circle cx="12" cy="10" r="3"/>
          </svg>
        </template>
      </van-cell>
      <van-cell title="优惠券" is-link>
        <template #icon>
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18" style="margin-right:8px">
            <rect x="2" y="4" width="20" height="16" rx="2"/><path d="M2 10h20"/>
          </svg>
        </template>
      </van-cell>
    </van-cell-group>

    <van-cell-group inset class="menu-group">
      <van-cell title="关于我们" is-link>
        <template #icon>
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18" style="margin-right:8px">
            <circle cx="12" cy="12" r="10"/><path d="M12 16v-4"/><path d="M12 8h.01"/>
          </svg>
        </template>
      </van-cell>
      <van-cell title="联系客服" is-link>
        <template #icon>
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="18" height="18" style="margin-right:8px">
            <path d="M21 15a2 2 0 0 1-2 2H7l-4 4V5a2 2 0 0 1 2-2h14a2 2 0 0 1 2 2z"/>
          </svg>
        </template>
      </van-cell>
    </van-cell-group>

    <div class="logout-wrap" v-if="userInfo.id">
      <van-button block plain round type="danger" @click="handleLogout">退出登录</van-button>
    </div>
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import { useRouter } from 'vue-router'
import { showDialog } from 'vant'

const router = useRouter()
const userInfo = ref({})

onMounted(() => {
  const stored = localStorage.getItem('user_info')
  if (stored) {
    try {
      userInfo.value = JSON.parse(stored)
    } catch {}
  }
})

const handleLogout = async () => {
  try {
    await showDialog({ title: '提示', message: '确定退出登录吗？' })
    localStorage.removeItem('user_token')
    localStorage.removeItem('user_info')
    router.replace('/login')
  } catch {}
}
</script>

<style scoped>
.mine-page {
  height: 100%;
  background: var(--bg);
  overflow-y: auto;
  padding-bottom: 80px;
}
.user-card {
  background: var(--brand);
  padding: 40px 20px 28px;
  display: flex;
  align-items: center;
  gap: 14px;
}
.avatar-placeholder {
  width: 60px;
  height: 60px;
  border-radius: 50%;
  background: rgba(255,255,255,.2);
  display: grid;
  place-items: center;
  color: #fff;
}
.user-info {
  color: #fff;
}
.nickname {
  font-size: 18px;
  font-weight: 700;
}
.user-id {
  font-size: 12px;
  opacity: .7;
  margin-top: 4px;
}

.menu-group {
  margin-top: 12px;
}
.logout-wrap {
  padding: 24px 16px;
}
</style>
