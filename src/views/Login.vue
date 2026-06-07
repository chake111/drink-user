<template>
  <div class="login-page">
    <div class="login-top">
      <div class="login-logo">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="36" height="36">
          <path d="M6 2h12l-1 7a5 5 0 0 1-10 0L6 2Z"/>
          <path d="M6 8H4a2 2 0 0 0 0 4h2"/>
          <path d="M5 22h14"/>
        </svg>
      </div>
      <h2 class="login-title">茶优选</h2>
      <p class="login-sub">每一杯，都是精选</p>
    </div>
    <div class="login-body">
      <div class="login-card">
        <h3>登录 / 注册</h3>
        <van-field
          v-model="account"
          label="账号"
          placeholder="请输入用户账号"
          :rules="[{ required: true, message: '请输入账号' }]"
        />
        <div style="margin-top: 20px">
          <van-button type="primary" block round @click="handleLogin" :loading="loading">
            登录
          </van-button>
        </div>
        <div class="login-divider">
          <span>其他方式</span>
        </div>
        <van-button block round plain class="wx-btn" @click="handleWxLogin">
          <template #icon>
            <svg viewBox="0 0 24 24" width="18" height="18">
              <path d="M8.5 11a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3zm5 0a1.5 1.5 0 1 0 0-3 1.5 1.5 0 0 0 0 3zm-1 7c-3.5 0-6.5-2.5-6.5-5.5S9 7 12.5 7 19 9.5 19 12.5c0 1-.3 2-.8 2.8l.8 2.7-2.5-1.3c-.8.5-1.7.8-2.7.8l-.3-.5z" fill="#07C160"/>
            </svg>
          </template>
          微信登录
        </van-button>
      </div>
    </div>
    <p class="login-foot">登录即代表同意《用户协议》和《隐私政策》</p>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRouter } from 'vue-router'
import { showToast } from 'vant'
import request from '../utils/request'

const router = useRouter()
const account = ref('')
const loading = ref(false)

const handleLogin = async () => {
  if (!account.value.trim()) {
    showToast('请输入账号')
    return
  }
  loading.value = true
  try {
    const res = await request.post('/user/login', { account: account.value })
    localStorage.setItem('user_token', res.data.token)
    localStorage.setItem('user_info', JSON.stringify(res.data))
    showToast('登录成功')
    router.replace('/home')
  } catch (err) {
    showToast(err.message || '登录失败')
  } finally {
    loading.value = false
  }
}

const handleWxLogin = () => {
  showToast('微信登录功能开发中')
}
</script>

<style scoped>
.login-page {
  min-height: 100vh;
  background: var(--brand-bg);
  display: flex;
  flex-direction: column;
}
.login-top {
  padding: 60px 32px 40px;
  text-align: center;
}
.login-logo {
  width: 72px;
  height: 72px;
  border-radius: 20px;
  background: var(--brand);
  color: #fff;
  display: grid;
  place-items: center;
  margin: 0 auto 20px;
}
.login-title {
  font-size: 24px;
  font-weight: 700;
  color: var(--ink);
  margin-bottom: 6px;
}
.login-sub {
  font-size: 14px;
  color: var(--ink-3);
}
.login-body {
  flex: 1;
  padding: 0 32px;
}
.login-card {
  background: var(--surface);
  border-radius: var(--radius-lg);
  padding: 28px 24px;
  box-shadow: 0 2px 20px rgba(0,0,0,.04);
}
.login-card h3 {
  font-size: 17px;
  font-weight: 600;
  margin-bottom: 20px;
  text-align: center;
}
.login-divider {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 20px 0;
  color: var(--ink-4);
  font-size: 12px;
}
.login-divider::before,
.login-divider::after {
  content: '';
  flex: 1;
  height: 1px;
  background: var(--line);
}
.wx-btn {
  color: #07C160 !important;
  border-color: #07C160 !important;
}
.login-foot {
  padding: 20px 32px 40px;
  text-align: center;
  font-size: 12px;
  color: var(--ink-4);
}
</style>
