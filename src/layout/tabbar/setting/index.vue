<template>
  <el-button size="small" icon="Refresh" circle @click="updateRefresh"></el-button>
  <el-button size="small" icon="FullScreen" circle @click="fullScreen"></el-button>
  <el-button size="small" icon="Setting" circle></el-button>
  <img :src="userStore.avatar" style="widows: 24px;height: 24px;margin:0 10px;border-radius: 50%;">
  <!-- 下拉菜单 -->
  <el-dropdown>
    <span class="el-dropdown-link">
      {{ userStore.username }}
      <el-icon class="el-icon--right">
        <arrow-down />
      </el-icon>
    </span>
    <template #dropdown>
      <el-dropdown-menu>
        <el-dropdown-item @click="logout">退出登录</el-dropdown-item>
      </el-dropdown-menu>
    </template>
  </el-dropdown>
</template>

<script setup lang='ts'>

// 获取骨架的小仓库
import useLayOutSettingStore from '@/store/modules/setting'
// 获取用户相关的小仓库
import useUserStore from '@/store/modules/user';
import { useRouter, useRoute } from 'vue-router';

let LayOutSettingStore = useLayOutSettingStore()
let userStore = useUserStore()
let $router = useRouter()
let $route = useRoute()

const updateRefresh = () => {
  LayOutSettingStore.refresh = !LayOutSettingStore.refresh
}

const fullScreen = () => {
  // console.log(document.fullscreenElement);
  let full = document.fullscreenElement

  if (!full) {
    document.documentElement.requestFullscreen()
  } else {
    document.exitFullscreen()
  }

}

const logout = async () => {
  // 第一件事：需要向服务器发请求[退出登录请求]
  // 第二件事：仓库当中用于相关的数据清空[token|username|avatar]
  // 第三件事：跳转到登录页
  await userStore.userLogout()
  $router.push({ path: '/login', query: { redirect: $route.path } })
}
</script>

<script lang="ts">
export default {
  name: 'Setting'
}
</script>

<style scoped></style>