<template>
  <div class="main-layout">
    <div class="main-content">
      <router-view />
    </div>
    <van-tabbar v-model="active" route active-color="#A6643C" inactive-color="#999">
      <van-tabbar-item to="/home" icon="wap-home-o">首页</van-tabbar-item>
      <van-tabbar-item to="/orders" icon="orders-o">订单</van-tabbar-item>
      <van-tabbar-item to="/mine" icon="user-o">我的</van-tabbar-item>
    </van-tabbar>
  </div>
</template>

<script setup>
import { ref } from 'vue'
import { useRoute } from 'vue-router'

const route = useRoute()
const active = ref(0)
</script>

<style scoped>
.main-layout {
  display: flex;
  flex-direction: column;
  height: 100vh;
  height: 100dvh;
  overflow: hidden;
}
.main-content {
  flex: 1;
  overflow: hidden;
}
</style>
