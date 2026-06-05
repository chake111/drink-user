<template>
  <div class="orders-page">
    <div class="page-title">我的订单</div>
    <van-tabs v-model:active="activeTab" sticky>
      <van-tab title="全部" name="all" />
      <van-tab title="待接单" name="1" />
      <van-tab title="制作中" name="2" />
      <van-tab title="待取餐" name="3" />
      <van-tab title="已完成" name="4" />
    </van-tabs>
    <div class="order-list">
      <div v-for="order in filteredOrders" :key="order.id" class="order-card">
        <div class="order-header">
          <span class="order-no">{{ order.orderNo }}</span>
          <span class="order-status" :class="'status-' + order.status">{{ statusText(order.status) }}</span>
        </div>
        <div v-for="item in order.details" :key="item.drinkName" class="order-item">
          <span class="item-name">{{ item.drinkName }}</span>
          <span class="item-spec">{{ item.spec }}</span>
          <span class="item-qty">x{{ item.quantity }}</span>
          <span class="item-price">¥{{ item.price }}</span>
        </div>
        <div class="order-footer">
          <span class="order-total">合计: ¥{{ order.amount }}</span>
          <van-button v-if="order.status === 1" size="small" plain type="danger" @click="cancelOrder(order)">取消订单</van-button>
        </div>
      </div>
      <van-empty v-if="filteredOrders.length === 0" description="暂无订单" />
    </div>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { showToast, showDialog } from 'vant'
import request from '../utils/request'

const activeTab = ref('all')
const orders = ref([])

const statusText = (s) => {
  const map = { 1: '待接单', 2: '制作中', 3: '待取餐', 4: '已完成', 5: '已取消' }
  return map[s] || '未知'
}

const filteredOrders = computed(() => {
  if (activeTab.value === 'all') return orders.value
  return orders.value.filter(o => String(o.status) === activeTab.value)
})

const fetchOrders = async () => {
  try {
    const res = await request.get('/user/order/list')
    orders.value = res.data || []
  } catch {
    orders.value = []
  }
}

const cancelOrder = async (order) => {
  try {
    await showDialog({ title: '确认取消', message: '确定要取消该订单吗？' })
    await request.post(`/user/order/cancel/${order.id}`)
    showToast('已取消')
    fetchOrders()
  } catch {
    // 用户取消或请求失败
  }
}

onMounted(() => {
  fetchOrders()
})
</script>

<style scoped>
.orders-page {
  height: 100%;
  display: flex;
  flex-direction: column;
  background: var(--bg);
}
.page-title {
  font-size: 18px;
  font-weight: 700;
  padding: 12px 16px;
  background: var(--surface);
}
.order-list {
  flex: 1;
  overflow-y: auto;
  padding: 12px 16px 80px;
}
.order-card {
  background: var(--surface);
  border-radius: var(--radius);
  padding: 14px 16px;
  margin-bottom: 12px;
}
.order-header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
}
.order-no {
  font-size: 13px;
  color: var(--ink-2);
}
.order-status {
  font-size: 13px;
  font-weight: 600;
}
.status-1 { color: var(--warn); }
.status-2 { color: var(--brand); }
.status-3 { color: #1890ff; }
.status-4 { color: var(--ok); }
.status-5 { color: var(--ink-4); }
.order-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 6px 0;
  font-size: 13px;
}
.item-name {
  flex: 1;
  font-weight: 500;
}
.item-spec {
  color: var(--ink-3);
  font-size: 12px;
}
.item-qty {
  color: var(--ink-3);
}
.item-price {
  font-weight: 600;
  color: var(--brand);
}
.order-footer {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-top: 10px;
  padding-top: 10px;
  border-top: 1px solid var(--line);
}
.order-total {
  font-size: 14px;
  font-weight: 700;
  color: var(--brand);
}
</style>
