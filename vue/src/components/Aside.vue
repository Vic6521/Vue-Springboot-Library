<template>
  <div>
    <el-menu
      style="width: 220px; min-height: calc(100vh - 64px);"
      :default-active="path"
      class="el-menu-vertical-demo"
      router
      background-color="#1f2937"
      text-color="#e5e7eb"
      active-text-color="#60a5fa"
    >
      <el-menu-item index="/dashboard">
        <svg class="icon" aria-hidden="true"><use xlink:href="#icondashboard"></use></svg>
        <span>展示板</span>
      </el-menu-item>

      <el-sub-menu index="2">
        <template #title>
          <svg class="icon" aria-hidden="true"><use xlink:href="#icon-mingpian"></use></svg>
          <span>个人信息</span>
        </template>
        <el-menu-item index="/person">
          <svg class="icon" aria-hidden="true"><use xlink:href="#icon-a-bianji1"></use></svg>
          <span>修改个人信息</span>
        </el-menu-item>
        <el-menu-item index="/password">
          <svg class="icon" aria-hidden="true"><use xlink:href="#icon-mima"></use></svg>
          <span>修改密码</span>
        </el-menu-item>
      </el-sub-menu>

      <el-menu-item index="/user" v-if="user.role == 1">
        <svg class="icon" aria-hidden="true"><use xlink:href="#iconreader"></use></svg>
        <span>读者管理</span>
      </el-menu-item>
      <el-menu-item index="/book" v-if="user.role == 1">
        <svg class="icon" aria-hidden="true"><use xlink:href="#iconbook"></use></svg>
        <span>书籍管理</span>
      </el-menu-item>
      <el-menu-item index="/lendrecord" v-if="user.role == 1">
        <svg class="icon" aria-hidden="true"><use xlink:href="#iconlend-record"></use></svg>
        <span>借阅管理</span>
      </el-menu-item>
      <el-menu-item index="/category" v-if="user.role == 1">
        <el-icon><Folder /></el-icon>
        <span>分类管理</span>
      </el-menu-item>
      <el-menu-item index="/reserve" v-if="user.role == 1">
        <el-icon><Calendar /></el-icon>
        <span>预约管理</span>
      </el-menu-item>
      <el-menu-item index="/fine" v-if="user.role == 1">
        <el-icon><Money /></el-icon>
        <span>罚款管理</span>
      </el-menu-item>

      <el-menu-item index="/book" v-if="user.role == 2">
        <svg class="icon" aria-hidden="true"><use xlink:href="#iconbook"></use></svg>
        <span>图书查询</span>
      </el-menu-item>
      <el-menu-item index="/lendrecord" v-if="user.role == 2">
        <svg class="icon" aria-hidden="true"><use xlink:href="#iconlend-record"></use></svg>
        <span>借阅信息</span>
      </el-menu-item>
      <el-menu-item index="/bookwithuser">
        <el-icon><grid /></el-icon>
        <span>借阅状态</span>
      </el-menu-item>
    </el-menu>
  </div>
</template>

<script>
export default {
  name: 'Aside',
  created() {
    const userStr = sessionStorage.getItem('user') || '{}'
    this.user = JSON.parse(userStr)
  },
  data() {
    return {
      user: {},
      path: this.$route.path
    }
  }
}
</script>

<style scoped>
.icon {
  width: 20px;
  height: 20px;
  margin-right: 10px;
}
</style>