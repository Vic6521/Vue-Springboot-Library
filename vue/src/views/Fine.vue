<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>罚款管理</h2>
        <p>用于展示逾期罚款记录、处理状态与统计汇总，适合作为数据库实践的扩展亮点。</p>
      </div>
      <div class="header-stats">
        <el-tag type="danger">未处理 {{ unpaidCount }} 条</el-tag>
        <el-tag type="success">已处理 {{ paidCount }} 条</el-tag>
      </div>
    </div>

    <el-card class="table-card" shadow="never">
      <div class="table-actions">
        <el-button type="primary" @click="load">刷新数据</el-button>
        <el-tag type="info">总记录 {{ total }} 条</el-tag>
      </div>

      <el-table :data="tableData" stripe border v-loading="loading">
        <el-table-column prop="readerName" label="读者" min-width="120" />
        <el-table-column prop="bookName" label="图书" min-width="150" />
        <el-table-column prop="overdueDays" label="逾期天数" width="110" />
        <el-table-column prop="fineAmount" label="罚款金额(元)" width="130" />
        <el-table-column prop="status" label="状态" width="120">
          <template #default="scope">
            <el-tag :type="scope.row.status === '已处理' ? 'success' : 'danger'">{{ scope.row.status || '未处理' }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="createTime" label="生成时间" min-width="160" />
        <el-table-column label="操作" width="160">
          <template #default="scope">
            <el-button size="small" type="primary" @click="markPaid(scope.row)" :disabled="scope.row.status === '已处理'">标记已处理</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
  </div>
</template>

<script>
import request from '@/utils/request'
import { ElMessage } from 'element-plus'

export default {
  name: 'Fine',
  data() {
    return {
      tableData: [],
      loading: false,
      total: 0,
      unpaidCount: 0,
      paidCount: 0
    }
  },
  created() {
    this.load()
  },
  methods: {
    load() {
      this.loading = true
      request.get('/fine').then(res => {
        this.tableData = res.data || []
        this.total = this.tableData.length
        this.unpaidCount = this.tableData.filter(i => i.status !== '已处理').length
        this.paidCount = this.tableData.filter(i => i.status === '已处理').length
      }).finally(() => {
        this.loading = false
      })
    },
    markPaid(row) {
      request.put('/fine', { ...row, status: '已处理' }).then(res => {
        if (res.code == 0) ElMessage.success('已标记处理')
        else ElMessage.error(res.msg)
        this.load()
      })
    }
  }
}
</script>

<style scoped>
.page-shell { padding: 18px; }
.page-header { display: flex; justify-content: space-between; gap: 16px; align-items: flex-start; margin-bottom: 16px; }
.page-header h2 { margin: 0 0 6px; }
.page-header p { margin: 0; color: #64748b; }
.header-stats { display: flex; gap: 10px; flex-wrap: wrap; }
.table-card { border-radius: 16px; }
.table-actions { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
@media (max-width: 768px) { .page-header { flex-direction: column; } }
</style>