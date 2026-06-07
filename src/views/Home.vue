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
        茶优选
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
        <div v-for="drink in drinks" :key="drink.id" class="drink-card" @click="showDrinkDetail(drink)">
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
              <div class="qty-control" @click.stop>
                <button v-if="getCartQty(drink.id) > 0" class="qty-btn" @click="decreaseCart(drink)">-</button>
                <span v-if="getCartQty(drink.id) > 0" class="qty-num">{{ getCartQty(drink.id) }}</span>
                <button class="qty-btn add" @click="quickAddToCart(drink)">+</button>
              </div>
            </div>
          </div>
        </div>
        <van-empty v-if="!loading && drinks.length === 0" description="暂无饮品" />
      </div>
    </div>

    <!-- 饮品详情弹窗 -->
    <van-action-sheet v-model:show="showDetailSheet" :title="detailDrink ? detailDrink.name : ''">
      <div class="detail-sheet" v-if="detailDrink">
        <div class="detail-img">
          <img v-if="detailDrink.image" :src="detailDrink.image" alt="" />
          <span v-else>☕</span>
        </div>
        <div class="detail-info">
          <div class="detail-name">{{ detailDrink.name }}</div>
          <div class="detail-desc">{{ detailDrink.description || '暂无描述' }}</div>
          <div class="detail-price"><small>¥</small>{{ detailDrink.price }}</div>
        </div>

        <!-- 口味选择 -->
        <div class="flavor-section" v-if="detailDrink.flavors && detailDrink.flavors.length">
          <div v-for="flavor in detailDrink.flavors" :key="flavor.name" class="flavor-group">
            <div class="flavor-label">{{ flavor.name }}</div>
            <div class="flavor-options">
              <span
                v-for="opt in parseFlavorValue(flavor.value)"
                :key="opt"
                class="flavor-option"
                :class="{ active: selectedFlavors[flavor.name] === opt }"
                @click="selectFlavor(flavor.name, opt)"
              >
                {{ opt }}
              </span>
            </div>
          </div>
        </div>

        <!-- 数量选择 -->
        <div class="detail-qty-row">
          <span class="detail-qty-label">数量</span>
          <div class="qty-control">
            <button class="qty-btn" @click="detailQty > 1 && detailQty--">-</button>
            <span class="qty-num">{{ detailQty }}</span>
            <button class="qty-btn add" @click="detailQty++">+</button>
          </div>
        </div>

        <!-- 加入购物车按钮 -->
        <div class="detail-footer">
          <van-button type="primary" block round @click="addToCartWithFlavor">
            加入购物车 ¥{{ (detailDrink.price * detailQty).toFixed(2) }}
          </van-button>
        </div>
      </div>
    </van-action-sheet>

    <!-- 购物车底部栏 -->
    <div class="cart-bar" v-if="cartItems.length > 0">
      <div class="cart-left" @click="showCartSheet = true">
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
        <div v-for="(item, index) in cartItems" :key="index" class="cart-item">
          <div class="cart-item-info">
            <span class="cart-item-name">{{ item.name }}</span>
            <span class="cart-item-spec" v-if="item.spec">{{ item.spec }}</span>
          </div>
          <span class="cart-item-price">¥{{ (item.price * item.qty).toFixed(2) }}</span>
          <div class="qty-control">
            <button class="qty-btn" @click="decreaseCartItem(index)">-</button>
            <span class="qty-num">{{ item.qty }}</span>
            <button class="qty-btn add" @click="item.qty++">+</button>
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
import { ref, computed, onMounted, reactive } from 'vue'
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

// 饮品详情弹窗相关
const showDetailSheet = ref(false)
const detailDrink = ref(null)
const detailQty = ref(1)
const selectedFlavors = reactive({})

const totalQty = computed(() => cartItems.value.reduce((sum, item) => sum + item.qty, 0))
const totalPrice = computed(() => cartItems.value.reduce((sum, item) => sum + item.price * item.qty, 0))

// 解析口味值（逗号分隔的字符串转数组）
const parseFlavorValue = (value) => {
  if (!value) return []
  return value.split(',').map(v => v.trim()).filter(v => v)
}

// 获取分类列表
const fetchCategories = async () => {
  try {
    const res = await request.get('/user/category/list')
    if (res.data && res.data.length) {
      categories.value = res.data
      activeCatId.value = res.data[0].id
      activeCatName.value = res.data[0].name
      fetchDrinks(res.data[0].id)
    }
  } catch {
    // 如果用户分类接口失败，尝试公共接口
    try {
      const res = await request.get('/category/list')
      if (res.data && res.data.length) {
        categories.value = res.data.filter(c => c.status === 1)
        if (categories.value.length) {
          activeCatId.value = categories.value[0].id
          activeCatName.value = categories.value[0].name
          fetchDrinks(categories.value[0].id)
        }
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
}

// 获取饮品列表
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

// 切换分类
const selectCategory = (id) => {
  activeCatId.value = id
  const cat = categories.value.find(c => c.id === id)
  activeCatName.value = cat ? cat.name : ''
  fetchDrinks(id)
}

// 显示饮品详情弹窗
const showDrinkDetail = async (drink) => {
  // 如果饮品没有口味数据，从接口获取完整信息
  if (!drink.flavors || drink.flavors.length === 0) {
    try {
      const res = await request.get(`/user/drink/${drink.id}`)
      detailDrink.value = res.data
    } catch {
      detailDrink.value = { ...drink, flavors: [] }
    }
  } else {
    detailDrink.value = { ...drink }
  }

  // 重置选择状态
  detailQty.value = 1
  Object.keys(selectedFlavors).forEach(key => delete selectedFlavors[key])

  // 自动选中每个口味的第一个选项
  if (detailDrink.value.flavors && detailDrink.value.flavors.length) {
    detailDrink.value.flavors.forEach(f => {
      const opts = parseFlavorValue(f.value)
      if (opts.length) {
        selectedFlavors[f.name] = opts[0]
      }
    })
  }

  showDetailSheet.value = true
}

// 选择口味
const selectFlavor = (name, value) => {
  selectedFlavors[name] = value
}

// 生成口味规格描述
const buildSpec = () => {
  const parts = Object.entries(selectedFlavors)
    .filter(([_, v]) => v)
    .map(([_, v]) => v)
  return parts.join('/')
}

// 获取购物车中某饮品的数量
const getCartQty = (drinkId) => {
  return cartItems.value
    .filter(i => i.id === drinkId)
    .reduce((sum, item) => sum + item.qty, 0)
}

// 快速加入购物车（不选口味，直接添加）
const quickAddToCart = (drink) => {
  // 如果饮品有口味选项，必须打开详情弹窗让用户选择口味
  if (drink.flavors && drink.flavors.length > 0) {
    showDrinkDetail(drink)
    return
  }
  // 检查是否已有相同饮品且无口味的条目
  const existing = cartItems.value.find(i => i.id === drink.id && !i.spec)
  if (existing) {
    existing.qty++
  } else {
    cartItems.value.push({
      id: drink.id,
      name: drink.name,
      price: drink.price,
      qty: 1,
      spec: ''
    })
  }
}

// 带口味加入购物车
const addToCartWithFlavor = () => {
  if (!detailDrink.value) return

  // 校验所有口味组是否都已选择
  if (detailDrink.value.flavors && detailDrink.value.flavors.length > 0) {
    const missing = detailDrink.value.flavors.filter(f => !selectedFlavors[f.name])
    if (missing.length > 0) {
      showToast(`请选择${missing.map(f => f.name).join('、')}`)
      return
    }
  }

  const spec = buildSpec()
  // 查找已有的相同饮品+相同口味的条目
  const existing = cartItems.value.find(
    i => i.id === detailDrink.value.id && i.spec === spec
  )
  if (existing) {
    existing.qty += detailQty.value
  } else {
    cartItems.value.push({
      id: detailDrink.value.id,
      name: detailDrink.value.name,
      price: detailDrink.value.price,
      qty: detailQty.value,
      spec: spec
    })
  }

  showSuccessToast('已加入购物车')
  showDetailSheet.value = false
}

// 减少购物车数量（按索引）
const decreaseCartItem = (index) => {
  const item = cartItems.value[index]
  if (item) {
    item.qty--
    if (item.qty <= 0) {
      cartItems.value.splice(index, 1)
    }
  }
}

// 减少购物车数量（按饮品ID，快速模式）
const decreaseCart = (drink) => {
  const existing = cartItems.value.find(i => i.id === drink.id && !i.spec)
  if (existing) {
    existing.qty--
    if (existing.qty <= 0) {
      const idx = cartItems.value.indexOf(existing)
      if (idx > -1) cartItems.value.splice(idx, 1)
    }
  } else {
    // 如果没有无口味的条目，减少最后一个该饮品的条目
    const lastItem = [...cartItems.value].reverse().find(i => i.id === drink.id)
    if (lastItem) {
      lastItem.qty--
      if (lastItem.qty <= 0) {
        const idx = cartItems.value.indexOf(lastItem)
        if (idx > -1) cartItems.value.splice(idx, 1)
      }
    }
  }
}

// 提交订单
const submitOrder = async () => {
  if (cartItems.value.length === 0) {
    showToast('购物车为空')
    return
  }
  try {
    const items = cartItems.value.map(item => ({
      drinkId: item.id,
      quantity: item.qty,
      spec: item.spec || ''
    }))
    await request.post('/user/order', {
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
  cursor: pointer;
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

/* 饮品详情弹窗 */
.detail-sheet {
  padding: 0 16px 16px;
  max-height: 70vh;
  overflow-y: auto;
}
.detail-img {
  width: 100%;
  height: 180px;
  border-radius: var(--radius);
  background: var(--brand-light);
  display: grid;
  place-items: center;
  font-size: 48px;
  overflow: hidden;
  margin-bottom: 16px;
}
.detail-img img { width: 100%; height: 100%; object-fit: cover; }
.detail-info {
  margin-bottom: 16px;
}
.detail-name {
  font-size: 18px;
  font-weight: 700;
  color: var(--ink);
  margin-bottom: 6px;
}
.detail-desc {
  font-size: 13px;
  color: var(--ink-3);
  margin-bottom: 8px;
  line-height: 1.5;
}
.detail-price {
  font-family: var(--font-en);
  font-size: 22px;
  font-weight: 700;
  color: var(--brand);
}
.detail-price small { font-size: 14px; font-weight: 400; }

/* 口味选择 */
.flavor-section {
  margin-bottom: 16px;
}
.flavor-group {
  margin-bottom: 12px;
}
.flavor-label {
  font-size: 13px;
  font-weight: 600;
  color: var(--ink);
  margin-bottom: 8px;
}
.flavor-options {
  display: flex;
  flex-wrap: wrap;
  gap: 8px;
}
.flavor-option {
  padding: 6px 14px;
  border-radius: 16px;
  font-size: 12px;
  background: var(--bg);
  color: var(--ink-2);
  cursor: pointer;
  transition: .15s;
  border: 1px solid transparent;
}
.flavor-option.active {
  background: var(--brand-light);
  color: var(--brand);
  border-color: var(--brand);
  font-weight: 600;
}

/* 数量选择行 */
.detail-qty-row {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 12px 0;
  border-top: 1px solid var(--line);
  border-bottom: 1px solid var(--line);
  margin-bottom: 16px;
}
.detail-qty-label {
  font-size: 14px;
  font-weight: 600;
  color: var(--ink);
}

.detail-footer {
  padding-top: 8px;
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
  cursor: pointer;
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
.cart-item-info {
  flex: 1;
  min-width: 0;
}
.cart-item-name {
  font-size: 14px;
  font-weight: 500;
  display: block;
}
.cart-item-spec {
  font-size: 11px;
  color: var(--ink-3);
  display: block;
  margin-top: 2px;
}
.cart-item-price {
  font-size: 14px;
  font-weight: 600;
  color: var(--brand);
  margin-right: 8px;
  white-space: nowrap;
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
