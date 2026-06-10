<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>修改密码</h2>
        <p>支持旧密码校验和双重确认，保证账号安全。</p>
      </div>
    </div>

    <el-card class="password-card" shadow="never">
      <el-form ref="form" :model="form" :rules="rules" label-width="110px" class="password-form">
        <el-form-item label="旧密码" prop="password2">
          <el-input v-model="form.password2" type="password" show-password autocomplete="off" />
        </el-form-item>
        <el-form-item label="新密码" prop="password">
          <el-input v-model="form2.password" type="password" show-password autocomplete="off" />
        </el-form-item>
        <el-form-item label="确认新密码" prop="checkpassword">
          <el-input v-model="form.checkpassword" type="password" show-password autocomplete="off" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="submitForm">提交</el-button>
          <el-button @click="resetForm('form')">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>
  </div>
</template>

<script>
import request from '../utils/request'
import { ElMessage } from 'element-plus'

export default {
  name: 'Password',
  data() {
    const validatePass2 = (rule, value, callback) => {
      if (value === '') return callback(new Error('请输入旧密码'))
      if (this.form.password2 !== this.form.truepassword) return callback(new Error('密码错误'))
      callback()
    }
    const validatePass = (rule, value, callback) => {
      if (value === '') return callback(new Error('请输入新密码'))
      callback()
    }
    const validatePass3 = (rule, value, callback) => {
      if (value === '') return callback(new Error('请再次输入密码'))
      if (value !== this.form2.password) return callback(new Error('两次输入密码不匹配'))
      callback()
    }
    return {
      form: { password2: '', checkpassword: '', truepassword: '' },
      form2: { password: '', id: 0 },
      rules: {
        password: [{ validator: validatePass, trigger: 'blur', required: true }],
        checkpassword: [{ validator: validatePass3, trigger: 'blur', required: true }],
        password2: [{ validator: validatePass2, trigger: 'blur', required: true }]
      }
    }
  },
  created() {
    const user = JSON.parse(sessionStorage.getItem('user'))
    this.form.truepassword = user.password
    this.form2.id = user.id
  },
  methods: {
    submitForm() {
      this.$refs['form'].validate((valid) => {
        if (!valid) return
        request.put('/user', this.form2).then(res => {
          if (res.code == 0) {
            ElMessage.success('密码修改成功,请重新登录')
            sessionStorage.removeItem('user')
            this.$router.push('/login')
          } else {
            ElMessage.error(res.msg)
          }
        })
      })
    },
    resetForm(formName) {
      this.$refs[formName].resetFields()
    }
  }
}
</script>

<style scoped>
.page-shell { padding: 18px; }
.page-header { margin-bottom: 16px; }
.page-header h2 { margin: 0 0 6px; }
.page-header p { margin: 0; color: #64748b; }
.password-card { max-width: 640px; border-radius: 18px; }
.password-form { max-width: 520px; }
</style>