<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>预约管理</h2>
        <p>用于展示图书预约、预约状态和取书提醒，是数据库实践中很典型的业务扩展功能。</p>
      </div>
    </div>

    <el-card class="filter-card" shadow="never">
      <div class="table-actions">
        <el-button type="primary" @click="openDialog()">新增预约</el-button>
        <el-tag type="info">共 {{ total }} 条预约</el-tag>
      </div>
      <el-table :data="tableData" stripe border v-loading="loading">
        <el-table-column prop="id" label="ID" width="90" />
        <el-table-column prop="readerName" label="读者" />
        <el-table-column prop="bookName" label="图书名称" />
        <el-table-column prop="reserveTime" label="预约时间" />
        <el-table-column prop="expireTime" label="到期时间" />
        <el-table-column prop="status" label="状态" width="120">
          <template #default="scope">
            <el-tag :type="statusType(scope.row.status)">{{ statusText(scope.row.status) }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180">
          <template #default="scope">
            <el-button size="small" @click="openDialog(scope.row)">编辑</el-button>
            <el-popconfirm title="确认删除?" @confirm="handleDelete(scope.row.id)">
              <template #reference>
                <el-button type="danger" size="small">删除</el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <el-dialog v-model="dialogVisible" title="预约信息" width="520px">
      <el-form :model="form" label-width="100px">
        <el-form-item label="读者名称"><el-input v-model="form.readerName" /></el-form-item>
        <el-form-item label="图书名称"><el-input v-model="form.bookName" /></el-form-item>
        <el-form-item label="预约时间"><el-date-picker v-model="form.reserveTime" type="datetime" value-format="YYYY-MM-DD HH:mm:ss" /></el-form-item>
        <el-form-item label="到期时间"><el-date-picker v-model="form.expireTime" type="datetime" value-format="YYYY-MM-DD HH:mm:ss" /></el-form-item>
        <el-form-item label="状态">
          <el-select v-model="form.status" style="width: 100%">
            <el-option label="待取书" value="待取书" />
            <el-option label="已取书" value="已取书" />
            <el-option label="已取消" value="已取消" />
          </el-select>
        </el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="save">确定</el-button>
      </template>
    </el-dialog>
  </div>
</template>

<script>
import request from '../utils/request'
import { ElMessage } from 'element-plus'

export default {
  name: 'Reserve',
  data() {
    return {
      tableData: [],
      loading: false,
      dialogVisible: false,
      form: {},
      total: 0
    }
  },
  created() {
    this.load()
  },
  methods: {
    load() {
      this.loading = true
      request.get('/reserve').then(res => {
        this.tableData = res.data || []
        this.total = this.tableData.length
      }).finally(() => {
        this.loading = false
      })
    },
    openDialog(row) {
      this.form = row ? JSON.parse(JSON.stringify(row)) : { status: '待取书' }
      this.dialogVisible = true
    },
    save() {
      const api = this.form.id ? request.put('/reserve', this.form) : request.post('/reserve', this.form)
      api.then(res => {
        if (res.code == 0) ElMessage.success(this.form.id ? '更新成功' : '新增成功')
        else ElMessage.error(res.msg)
        this.dialogVisible = false
        this.load()
      })
    },
    handleDelete(id) {
      request.delete('/reserve/' + id).then(res => {
        if (res.code == 0) ElMessage.success('删除成功')
        else ElMessage.error(res.msg)
        this.load()
      })
    },
    statusText(status) {
      return status || '待取书'
    },
    statusType(status) {
      if (status === '已取书') return 'success'
      if (status === '已取消') return 'info'
      return 'warning'
    }
  }
}
</script>

<style scoped>
.page-shell { padding: 18px; }
.page-header { margin-bottom: 16px; }
.page-header h2 { margin: 0 0 6px; }
.page-header p { margin: 0; color: #64748b; }
.filter-card { border-radius: 16px; }
.table-actions { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
</style>