<template>
  <div class="login-shell">
    <div class="login-decor">
      <div class="brand-badge">Library System</div>
      <h1>图书馆管理系统</h1>
      <p>面向管理员与读者的统一入口。支持图书借阅、记录管理、数据统计与个人中心，适合作为数据库实践大作业展示。</p>
      <div class="feature-list">
        <div>多角色登录</div>
        <div>数据统计面板</div>
        <div>借阅与归还流程</div>
      </div>
    </div>

    <el-form ref="form" :model="form" :rules="rules" class="login-card">
      <div class="login-title">
        <h2>系统登录</h2>
        <span>请输入账号密码与验证码</span>
      </div>

      <el-form-item prop="username">
        <el-input v-model="form.username" clearable placeholder="用户名">
          <template #prefix>
            <el-icon class="el-input__icon"><User /></el-icon>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item prop="password">
        <el-input v-model="form.password" clearable show-password placeholder="密码">
          <template #prefix>
            <el-icon class="el-input__icon"><Lock /></el-icon>
          </template>
        </el-input>
      </el-form-item>

      <el-form-item>
        <div class="captcha-row">
          <el-input v-model="form.validCode" class="captcha-input" placeholder="请输入验证码"></el-input>
          <ValidCode @input="createValidCode" class="captcha-box" />
        </div>
      </el-form-item>

      <el-form-item>
        <el-button type="primary" class="login-btn" @click="login">登 录</el-button>
      </el-form-item>

      <div class="login-footer">
        <el-button link type="primary" @click="$router.push('/register')">前往注册</el-button>
      </div>
    </el-form>
  </div>
</template>

<script>
import request from '../utils/request'
import { ElMessage } from 'element-plus'
import ValidCode from '../components/Validate'

export default {
  name: 'Login',
  components: {
    ValidCode
  },
  data() {
    return {
      validCode: '',
      form: {},
      rules: {
        username: [{ required: true, message: '请输入用户名', trigger: 'blur' }],
        password: [{ required: true, message: '请输入密码', trigger: 'blur' }]
      }
    }
  },
  methods: {
    createValidCode(data) {
      this.validCode = data
    },
    login() {
      this.$refs['form'].validate((valid) => {
        if (!valid) return
        if (!this.form.validCode) {
          ElMessage.error('请填写验证码')
          return
        }
        if (this.form.validCode.toLowerCase() !== this.validCode.toLowerCase()) {
          ElMessage.error('验证码错误')
          return
        }
        request.post('user/login', this.form).then(res => {
          if (res.code == 0) {
            ElMessage.success('登录成功')
            sessionStorage.setItem('user', JSON.stringify(res.data))
            this.$router.push('/dashboard')
          } else {
            ElMessage.error(res.msg)
          }
        })
      })
    }
  }
}
</script>

<style scoped>
.login-shell {
  min-height: 100vh;
  display: grid;
  grid-template-columns: 1.1fr 0.9fr;
  background:
    radial-gradient(circle at top left, rgba(59, 130, 246, 0.18), transparent 34%),
    linear-gradient(180deg, #f8fafc 0%, #eef2ff 100%);
  padding: 40px;
  gap: 30px;
  align-items: center;
}

.login-decor {
  padding: 40px;
  color: #0f172a;
}

.brand-badge {
  display: inline-flex;
  align-items: center;
  padding: 8px 14px;
  border-radius: 999px;
  background: rgba(37, 99, 235, 0.1);
  color: #2563eb;
  font-weight: 700;
  margin-bottom: 18px;
}

.login-decor h1 {
  font-size: 44px;
  line-height: 1.2;
  margin: 0 0 16px;
}

.login-decor p {
  max-width: 560px;
  line-height: 1.8;
  color: #475569;
  margin: 0 0 24px;
}

.feature-list {
  display: flex;
  flex-wrap: wrap;
  gap: 12px;
}

.feature-list div {
  padding: 10px 14px;
  border-radius: 12px;
  background: #fff;
  box-shadow: 0 10px 25px rgba(15, 23, 42, 0.06);
  color: #334155;
}

.login-card {
  max-width: 430px;
  width: 100%;
  justify-self: end;
  padding: 34px;
  border-radius: 22px;
  background: rgba(255, 255, 255, 0.95);
  box-shadow: 0 20px 50px rgba(15, 23, 42, 0.12);
  border: 1px solid rgba(148, 163, 184, 0.18);
}

.login-title h2 {
  margin: 0;
  font-size: 28px;
}

.login-title span {
  color: #64748b;
  font-size: 13px;
}

.captcha-row {
  display: flex;
  gap: 12px;
  width: 100%;
}

.captcha-input {
  flex: 1;
}

.captcha-box {
  width: 130px;
  flex-shrink: 0;
}

.login-btn {
  width: 100%;
  height: 44px;
  border-radius: 12px;
}

.login-footer {
  text-align: center;
}

@media (max-width: 960px) {
  .login-shell {
    grid-template-columns: 1fr;
    padding: 20px;
  }

  .login-decor {
    padding: 10px 10px 0;
  }

  .login-decor h1 {
    font-size: 32px;
  }

  .login-card {
    justify-self: stretch;
    max-width: none;
  }
}
</style>