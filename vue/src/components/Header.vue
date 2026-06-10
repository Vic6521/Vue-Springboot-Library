<template>
  <div class="topbar">
    <div class="brand">
      <img :src="imgUrl" class="icon" alt="logo" />
      <div>
        <div class="title">图书馆管理系统</div>
        <div class="subtitle">Database Practice Project</div>
      </div>
    </div>

    <div class="grow"></div>

    <el-dropdown>
      <span class="el-dropdown-link user-link">
        {{ user.nickName }}
        <el-icon class="el-icon--right"><arrow-down /></el-icon>
      </span>
      <template #dropdown>
        <el-dropdown-menu>
          <el-dropdown-item @click="exit">退出系统</el-dropdown-item>
        </el-dropdown-menu>
      </template>
    </el-dropdown>
  </div>
</template>

<script>
import { ElMessage } from 'element-plus'

export default {
  name: 'Header',
  data() {
    return {
      user: {},
      imgUrl: require('../assets/icon/login.png')
    }
  },
  created() {
    const userStr = sessionStorage.getItem('user') || '{}'
    this.user = JSON.parse(userStr)
  },
  methods: {
    exit() {
      sessionStorage.removeItem('user')
      this.$router.push('/login')
      ElMessage.success('退出系统成功')
    }
  }
}
</script>

<style scoped>
.topbar {
  height: 64px;
  padding: 0 20px;
  display: flex;
  align-items: center;
  border-bottom: 1px solid #e5e7eb;
  background: #fff;
}
.brand {
  display: flex;
  align-items: center;
  gap: 12px;
  width: 260px;
}
.icon { width: 40px; height: 40px; }
.title { font-weight: 800; color: #0f172a; }
.subtitle { font-size: 12px; color: #64748b; margin-top: 2px; }
.grow { flex: 1; }
.user-link { display: inline-flex; align-items: center; gap: 6px; cursor: pointer; color: #334155; }
</style>