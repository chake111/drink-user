<template>
  <div class="home-page">
    <!-- 顶部品牌栏 -->
    <div class="home-header">
      <div class="logo-sm">
        <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="16" height="16">
          <path d="M6 2h12l-1 7a5 5 0 0 1-10 0L6 2Z"/>
          <path d="M6 8H4a2 2 0 0 0 0 4h2"/>
          <path d="M5 22h14"/>
        </svg>
      </div>
      <span class="brand-text">
        茶咖优选
        <small>营业中</small>
      </span>
    </div>

    <!-- 主体：左右布局 -->
    <div class="main-body">
      <!-- 左侧品类 -->
      <div class="cat-sidebar">
        <div
          v-for="cat in categories"
          :key="cat.id"
          class="cat-item"
          :class="{ active: activeCatId === cat.id }"
          @click="selectCategory(cat.id)"
        >
          {{ cat.name }}
        </div>
      </div>

      <!-- 右侧饮品列表 -->
      <div class="drink-list">
        <div class="cat-section-title">{{ activeCatName }}</div>
        <div v-for="drink in drinks" :key="drink.id" class="drink-card">
          <div class="drink-img">
            <img v-if="drink.image" :src="drink.image" alt="" />
            <span v-else>☕</span>
          </div>
          <div class="drink-info">
            <div class="drink-name">{{ drink.name }}</div>
            <div class="drink-desc">{{ drink.description }}</div>
            <div class="drink-tags" v-if="drink.flavors && drink.flavors.length">
              <span class="drink-tag" v-for="f in drink.flavors.slice(0, 2)" :key="f.name">{{ f.name }}</span>
            </div>
            <div class="drink-bottom">
              <span class="drink-price"><small>¥</small>{{ drink.price }}</span>
              <div class="qty-control">
                <button v-if="getCartQty(drink.id) > 0" class="qty-btn" @click="decreaseCart(drink)">-</button>
                <span v-if="getCartQty(drink.id) > 0" class="qty-num">{{ getCartQty(drink.id) }}</span>
                <button class="qty-btn add" @click="addToCart(drink)">+</button>
              </div>
            </div>
          </div>
        </div>
        <van-empty v-if="!loading && drinks.length === 0" description="暂无饮品" />
      </div>
    </div>

    <!-- 购物车底部栏 -->
    <div class="cart-bar" v-if="cartItems.length > 0">
      <div class="cart-left">
        <div class="cart-icon">
          <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" width="22" height="22">
            <path d="M6 2L3 6v14a2 2 0 0 0 2 2h14a2 2 0 0 0 2-2V6l-3-4z"/><line x1="3" y1="6" x2="21" y2="6"/><path d="M16 10a4 4 0 0 1-8 0"/>
          </svg>
          <span class="cart-badge">{{ totalQty }}</span>
        </div>
        <div class="cart-info">
          <span class="cart-price">¥{{ totalPrice.toFixed(2) }}</span>
        </div>
      </div>
      <van-button type="primary" round size="small" @click="showCartSheet = true">去下单</van-button>
    </div>

    <!-- 购物车详情弹窗 -->
    <van-action-sheet v-model:show="showCartSheet" title="购物车">
      <div class="cart-sheet">
        <div v-for="item in cartItems" :key="item.id" class="cart-item">
          <span class="cart-item-name">{{ item.name }}</span>
          <span class="cart-item-price">¥{{ (item.price * item.qty).toFixed(2) }}</span>
          <div class="qty-control">
            <button class="qty-btn" @click="decreaseCart(item)">-</button>
            <span class="qty-num">{{ item.qty }}</span>
            <button class="qty-btn add" @click="addToCart(item)">+</button>
          </div>
        </div>
        <div class="cart-sheet-footer">
          <div class="pickup-type">
            <span>取餐方式：</span>
            <van-radio-group v-model="pickupType" direction="horizontal">
              <van-radio name="自取">自取</van-radio>
              <van-radio name="外送">外送</van-radio>
            </van-radio-group>
          </div>
          <van-button type="primary" block round @click="submitOrder">提交订单 ¥{{ totalPrice.toFixed(2) }}</van-button>
        </div>
      </div>
    </van-action-sheet>
  </div>
</template>

<script setup>
import { ref, computed, onMounted } from 'vue'
import { showToast, showSuccessToast } from 'vant'
import { useRouter } from 'vue-router'
import request from '../utils/request'

const router = useRouter()
const categories = ref([])
const drinks = ref([])
const activeCatId = ref(null)
const activeCatName = ref('推荐')
const loading = ref(false)
const cartItems = ref([])
const showCartSheet = ref(false)
const pickupType = ref('自取')

const totalQty = computed(() => cartItems.value.reduce((sum, item) => sum + item.qty, 0))
const totalPrice = computed(() => cartItems.value.reduce((sum, item) => sum + item.price * item.qty, 0))

const fetchCategories = async () => {
  try {
    const res = await request.get('/category/list')
    if (res.data && res.data.length) {
      categories.value = res.data
      activeCatId.value = res.data[0].id
      activeCatName.value = res.data[0].name
      fetchDrinks(res.data[0].id)
    }
  } catch {
    categories.value = [
      { id: 1, name: '推荐' },
      { id: 2, name: '奶茶' },
      { id: 3, name: '咖啡' }
    ]
    activeCatId.value = 1
    activeCatName.value = '推荐'
  }
}

const fetchDrinks = async (categoryId) => {
  loading.value = true
  try {
    const res = await request.get('/user/drink/list', { params: { categoryId } })
    drinks.value = res.data || []
  } catch {
    drinks.value = []
  } finally {
    loading.value = false
  }
}

const selectCategory = (id) => {
  activeCatId.value = id
  const cat = categories.value.find(c => c.id === id)
  activeCatName.value = cat ? cat.name : ''
  fetchDrinks(id)
}

const getCartQty = (drinkId) => {
  const item = cartItems.value.find(i => i.id === drinkId)
  return item ? item.qty : 0
}

const addToCart = (drink) => {
  const existing = cartItems.value.find(i => i.id === drink.id)
  if (existing) {
    existing.qty++
  } else {
    cartItems.value.push({ id: drink.id, name: drink.name, price: drink.price, qty: 1 })
  }
}

const decreaseCart = (drink) => {
  const existing = cartItems.value.find(i => i.id === drink.id)
  if (existing) {
    existing.qty--
    if (existing.qty <= 0) {
      cartItems.value = cartItems.value.filter(i => i.id !== drink.id)
    }
  }
}

const submitOrder = async () => {
  if (cartItems.value.length === 0) {
    showToast('购物车为空')
    return
  }
  try {
    const items = cartItems.value.map(item => ({
      drinkId: item.id,
      quantity: item.qty,
      spec: ''
    }))
    const res = await request.post('/user/order', {
      pickupType: pickupType.value,
      items
    })
    showSuccessToast('下单成功')
    cartItems.value = []
    showCartSheet.value = false
    router.push('/orders')
  } catch (err) {
    showToast(err.message || '下单失败')
  }
}

onMounted(() => {
  fetchCategories()
})
</script>

<style scoped>
.home-page {
  display: flex;
  flex-direction: column;
  height: 100%;
}
.home-header {
  padding: 8px 16px 12px;
  display: flex;
  align-items: center;
  gap: 10px;
  flex-shrink: 0;
  background: var(--surface);
}
.logo-sm {
  width: 30px;
  height: 30px;
  border-radius: 8px;
  background: var(--brand);
  color: #fff;
  display: grid;
  place-items: center;
  flex-shrink: 0;
}
.brand-text {
  font-family: var(--font-en);
  font-size: 15px;
  font-weight: 700;
}
.brand-text small {
  display: block;
  font-family: var(--font-cn);
  font-size: 10px;
  font-weight: 400;
  color: var(--ink-3);
}
.main-body {
  flex: 1;
  display: flex;
  overflow: hidden;
}
.cat-sidebar {
  width: 88px;
  flex-shrink: 0;
  background: #FAFAFA;
  overflow-y: auto;
  scrollbar-width: none;
}
.cat-sidebar::-webkit-scrollbar { display: none; }
.cat-item {
  padding: 14px 0;
  text-align: center;
  font-size: 12px;
  color: var(--ink-2);
  position: relative;
  cursor: pointer;
  transition: .15s;
}
.cat-item.active {
  background: var(--surface);
  color: var(--ink);
  font-weight: 600;
}
.cat-item.active::before {
  content: '';
  position: absolute;
  left: 0;
  top: 20%;
  bottom: 20%;
  width: 3px;
  border-radius: 0 3px 3px 0;
  background: var(--brand);
}
.drink-list {
  flex: 1;
  overflow-y: auto;
  padding: 0 12px 100px;
}
.drink-list::-webkit-scrollbar { display: none; }
.cat-section-title {
  font-size: 14px;
  font-weight: 600;
  color: var(--ink);
  padding: 14px 0 8px;
  position: sticky;
  top: 0;
  background: var(--surface);
  z-index: 1;
}
.drink-card {
  display: flex;
  gap: 10px;
  padding: 10px 0;
  border-bottom: 1px solid var(--line);
}
.drink-card:last-child { border-bottom: 0; }
.drink-img {
  width: 72px;
  height: 72px;
  border-radius: 10px;
  background: var(--brand-light);
  flex-shrink: 0;
  display: grid;
  place-items: center;
  font-size: 24px;
  overflow: hidden;
}
.drink-img img { width: 100%; height: 100%; object-fit: cover; }
.drink-info {
  flex: 1;
  min-width: 0;
  display: flex;
  flex-direction: column;
}
.drink-name {
  font-size: 14px;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 2px;
}
.drink-desc {
  font-size: 11px;
  color: var(--ink-3);
  margin-bottom: 4px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
.drink-tags {
  display: flex;
  gap: 4px;
  margin-bottom: 6px;
}
.drink-tag {
  font-size: 10px;
  padding: 2px 6px;
  border-radius: 4px;
  background: var(--brand-light);
  color: var(--brand);
  font-weight: 500;
}
.drink-bottom {
  display: flex;
  align-items: center;
  justify-content: space-between;
  margin-top: auto;
}
.drink-price {
  font-family: var(--font-en);
  font-size: 16px;
  font-weight: 700;
  color: var(--brand);
}
.drink-price small { font-size: 11px; font-weight: 400; }

.qty-control {
  display: flex;
  align-items: center;
  gap: 8px;
}
.qty-btn {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  border: 1px solid var(--line-2);
  background: var(--surface);
  color: var(--ink-2);
  display: grid;
  place-items: center;
  cursor: pointer;
  font-size: 14px;
}
.qty-btn.add {
  background: var(--brand);
  border-color: var(--brand);
  color: #fff;
}
.qty-num {
  font-size: 14px;
  font-weight: 600;
  min-width: 16px;
  text-align: center;
}

/* 购物车底部栏 */
.cart-bar {
  position: fixed;
  bottom: 50px;
  left: 0;
  right: 0;
  background: var(--surface);
  border-top: 1px solid var(--line);
  padding: 8px 16px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  z-index: 100;
}
.cart-left {
  display: flex;
  align-items: center;
  gap: 12px;
}
.cart-icon {
  position: relative;
  width: 44px;
  height: 44px;
  border-radius: 50%;
  background: var(--brand);
  color: #fff;
  display: grid;
  place-items: center;
}
.cart-badge {
  position: absolute;
  top: -4px;
  right: -4px;
  background: var(--warn);
  color: #fff;
  font-size: 10px;
  min-width: 18px;
  height: 18px;
  border-radius: 9px;
  display: grid;
  place-items: center;
  font-weight: 600;
}
.cart-price {
  font-size: 18px;
  font-weight: 700;
  color: var(--brand);
}

/* 购物车弹窗 */
.cart-sheet {
  padding: 0 16px 16px;
  max-height: 50vh;
  overflow-y: auto;
}
.cart-item {
  display: flex;
  align-items: center;
  gap: 12px;
  padding: 12px 0;
  border-bottom: 1px solid var(--line);
}
.cart-item-name {
  flex: 1;
  font-size: 14px;
  font-weight: 500;
}
.cart-item-price {
  font-size: 14px;
  font-weight: 600;
  color: var(--brand);
  margin-right: 8px;
}
.cart-sheet-footer {
  margin-top: 16px;
}
.pickup-type {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 16px;
  font-size: 14px;
}
</style>
