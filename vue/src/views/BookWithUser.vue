<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>借阅与续借管理</h2>
        <p>展示读者借阅历史、续借次数和最迟归还日期，是答辩中体现业务规则的重要页面。</p>
      </div>
    </div>

    <el-card class="filter-card" shadow="never">
      <el-form :inline="true" size="small">
        <el-form-item label="图书编号"><el-input v-model="search1" placeholder="请输入图书编号" clearable /></el-form-item>
        <el-form-item label="图书名称"><el-input v-model="search2" placeholder="请输入图书名称" clearable /></el-form-item>
        <el-form-item label="借阅者" v-if="user.role == 1"><el-input v-model="search3" placeholder="请输入借阅者昵称" clearable /></el-form-item>
        <el-form-item>
          <el-button type="primary" @click="load">查询</el-button>
          <el-button @click="clear">重置</el-button>
        </el-form-item>
      </el-form>
    </el-card>

    <el-card class="table-card" shadow="never">
      <div class="table-actions">
        <el-popconfirm v-if="user.role == 1" title="确认删除?" @confirm="deleteBatch">
          <template #reference>
            <el-button type="danger">批量删除</el-button>
          </template>
        </el-popconfirm>
        <el-tag type="info">总记录 {{ total }} 条</el-tag>
      </div>

      <el-table :data="tableData" stripe border v-loading="loading" @selection-change="handleSelectionChange">
        <el-table-column v-if="user.role == 1" type="selection" width="55" />
        <el-table-column prop="isbn" label="图书编号" sortable />
        <el-table-column prop="bookName" label="图书名称" />
        <el-table-column prop="nickName" label="借阅者" />
        <el-table-column prop="lendtime" label="借阅时间" />
        <el-table-column prop="deadtime" label="最迟归还日期" />
        <el-table-column prop="prolong" label="可续借次数" />
        <el-table-column fixed="right" label="操作" width="180">
          <template #default="scope">
            <el-button v-if="user.role == 1" size="small" @click="handleEdit(scope.row)">修改</el-button>
            <el-popconfirm v-if="user.role == 1" title="确认删除?" @confirm="handleDelete(scope.row)">
              <template #reference><el-button type="danger" size="small">删除</el-button></template>
            </el-popconfirm>
            <el-popconfirm v-if="user.role == 2" title="确认续借(续借一次延长30天)?" @confirm="handlereProlong(scope.row)" :disabled="scope.row.prolong == 0">
              <template #reference><el-button type="warning" size="small" :disabled="scope.row.prolong == 0">续借</el-button></template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>

      <div class="pagination-wrap">
        <el-pagination
          v-model:currentPage="currentPage"
          :page-sizes="[5, 10, 20]"
          :page-size="pageSize"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        />
      </div>
    </el-card>

    <el-dialog v-model="dialogVisible2" title="修改借阅信息" width="520px">
      <el-form :model="form" label-width="100px">
        <el-form-item label="图书编号"><el-input v-model="form.isbn" /></el-form-item>
        <el-form-item label="图书名称"><el-input v-model="form.bookName" /></el-form-item>
        <el-form-item label="借阅者"><el-input v-model="form.nickName" /></el-form-item>
        <el-form-item label="续借次数"><el-input v-model="form.prolong" /></el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible2 = false">取消</el-button>
        <el-button type="primary" @click="save">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import request from '../utils/request'
import { ElMessage } from 'element-plus'
import moment from 'moment'

export default {
  name: 'BookWithUser',
  created() {
    const userStr = sessionStorage.getItem('user') || '{}'
    this.user = JSON.parse(userStr)
    this.load()
  },
  data() {
    return {
      form: {},
      dialogVisible2: false,
      search1: '',
      search2: '',
      search3: '',
      total: 0,
      currentPage: 1,
      pageSize: 10,
      tableData: [],
      user: {},
      forms: [],
      loading: false
    }
  },
  methods: {
    handleSelectionChange(val) {
      this.forms = val
    },
    deleteBatch() {
      if (!this.forms.length) return ElMessage.warning('请选择数据！')
      request.post('bookwithuser/deleteRecords', this.forms).then(res => {
        if (res.code === '0' || res.code === 0) {
          ElMessage.success('批量删除成功')
          this.load()
        } else {
          ElMessage.error(res.msg)
        }
      })
    },
    load() {
      this.loading = true
      request.get('/bookwithuser', {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize,
          search1: this.search1,
          search2: this.search2,
          search3: this.user.role == 1 ? this.search3 : this.user.id
        }
      }).then(res => {
        this.tableData = res.data.records || []
        this.total = res.data.total || 0
      }).finally(() => {
        this.loading = false
      })
    },
    clear() {
      this.search1 = ''
      this.search2 = ''
      this.search3 = ''
      this.load()
    },
    handleDelete(row) {
      request.post('bookwithuser/deleteRecord', JSON.parse(JSON.stringify(row))).then(res => {
        if (res.code == 0) ElMessage.success('删除成功')
        else ElMessage.error(res.msg)
        this.load()
      })
    },
    handlereProlong(row) {
      const nowDate = new Date(row.deadtime)
      nowDate.setDate(nowDate.getDate() + 30)
      row.deadtime = moment(nowDate).format('yyyy-MM-DD HH:mm:ss')
      row.prolong = row.prolong - 1
      request.post('/bookwithuser', row).then(res => {
        if (res.code == 0) ElMessage.success('续借成功')
        else ElMessage.error(res.msg)
        this.load()
        this.dialogVisible2 = false
      })
    },
    save() {
      request.post('/bookwithuser', this.form).then(res => {
        if (res.code == 0) ElMessage.success('修改信息成功')
        else ElMessage.error(res.msg)
        this.load()
        this.dialogVisible2 = false
      })
    },
    handleEdit(row) {
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible2 = true
    },
    handleSizeChange(pageSize) {
      this.pageSize = pageSize
      this.load()
    },
    handleCurrentChange(pageNum) {
      this.currentPage = pageNum
      this.load()
    }
  }
}
</script>

<style scoped>
.page-shell { padding: 18px; }
.page-header { margin-bottom: 16px; }
.page-header h2 { margin: 0 0 6px; }
.page-header p { margin: 0; color: #64748b; }
.filter-card,.table-card { border-radius: 16px; margin-bottom: 16px; }
.table-actions { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.pagination-wrap { display: flex; justify-content: flex-end; margin-top: 16px; }
</style>