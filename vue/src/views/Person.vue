<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>个人信息</h2>
        <p>这里可以维护读者资料，便于展示用户中心和信息修改功能。</p>
      </div>
    </div>

    <el-card class="profile-card" shadow="never">
      <div class="profile-top">
        <div class="avatar-block">{{ (form.nickName || form.username || 'U').slice(0, 1) }}</div>
        <div>
          <div class="profile-name">{{ form.nickName || form.username }}</div>
          <div class="profile-role">{{ form.role == 1 ? '管理员' : '读者' }}</div>
        </div>
      </div>

      <el-form :model="form" ref="form" label-width="90px" class="profile-form">
        <el-form-item label="用户名"><el-input v-model="form.username" disabled /></el-form-item>
        <el-form-item label="姓名"><el-input v-model="form.nickName" /></el-form-item>
        <el-form-item label="权限">
          <el-tag :type="form.role == 1 ? 'danger' : 'success'">{{ form.role == 1 ? '管理员' : '读者' }}</el-tag>
        </el-form-item>
        <el-form-item label="电话号码"><el-input v-model="form.phone" /></el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="form.sex">
            <el-radio label="男">男</el-radio>
            <el-radio label="女">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="地址"><el-input v-model="form.address" type="textarea" :rows="3" /></el-form-item>
      </el-form>

      <div class="action-row">
        <el-button type="primary" @click="update">保存修改</el-button>
      </div>
    </el-card>
  </div>
</template>

<script>
import request from '@/utils/request'
import { ElMessage } from 'element-plus'

export default {
  name: 'Person',
  data() {
    return { form: {} }
  },
  created() {
    const str = sessionStorage.getItem('user') || '{}'
    this.form = JSON.parse(str)
  },
  methods: {
    update() {
      request.put('/user', this.form).then(res => {
        if (res.code === '0' || res.code === 0) {
          ElMessage.success('更新成功')
          sessionStorage.setItem('user', JSON.stringify(this.form))
          this.$emit('userInfo')
        } else {
          ElMessage.error(res.msg)
        }
      })
    }
  }
}
</script>

<style scoped>
.page-shell { padding: 18px; }
.page-header { margin-bottom: 16px; }
.page-header h2 { margin: 0 0 6px; }
.page-header p { margin: 0; color: #64748b; }
.profile-card { max-width: 780px; border-radius: 18px; }
.profile-top { display: flex; align-items: center; gap: 16px; margin-bottom: 20px; }
.avatar-block { width: 56px; height: 56px; border-radius: 50%; background: #2563eb; color: #fff; display: flex; align-items: center; justify-content: center; font-size: 24px; font-weight: 700; }
.profile-name { font-size: 20px; font-weight: 700; color: #111827; }
.profile-role { color: #64748b; margin-top: 4px; }
.profile-form { max-width: 640px; }
.action-row { text-align: center; margin-top: 20px; }
</style>