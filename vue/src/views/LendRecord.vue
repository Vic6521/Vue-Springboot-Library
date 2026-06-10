<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>借阅记录管理</h2>
        <p>集中展示借阅、归还、逾期、罚款与状态流转，便于在答辩时说明数据库事务与记录追踪。</p>
      </div>
      <div class="header-stats">
        <el-tag type="warning">未归还 {{ overdueCount }} 条</el-tag>
        <el-tag type="success">已归还 {{ returnedCount }} 条</el-tag>
        <el-tag type="info">总记录 {{ total }} 条</el-tag>
      </div>
    </div>

    <el-card class="filter-card" shadow="never">
      <el-form :inline="true" size="small">
        <el-form-item label="图书编号"><el-input v-model="search1" placeholder="请输入图书编号" clearable /></el-form-item>
        <el-form-item label="图书名称"><el-input v-model="search2" placeholder="请输入图书名称" clearable /></el-form-item>
        <el-form-item label="读者编号"><el-input v-model="search3" placeholder="请输入读者编号" clearable /></el-form-item>
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
        <el-table-column prop="bookname" label="图书名称" />
        <el-table-column prop="readerId" label="读者编号" sortable />
        <el-table-column prop="lendTime" label="借阅时间" sortable />
        <el-table-column prop="returnTime" label="归还时间" sortable />
        <el-table-column prop="status" label="状态">
          <template #default="scope">
            <el-tag :type="scope.row.status == 0 ? 'warning' : 'success'">{{ scope.row.status == 0 ? '未归还' : '已归还' }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="逾期/罚款" width="160">
          <template #default="scope">
            <el-tag v-if="isOverdue(scope.row)" type="danger">{{ overdueDays(scope.row) }} 天 / {{ calculateFine(scope.row) }} 元</el-tag>
            <el-tag v-else type="success">正常</el-tag>
          </template>
        </el-table-column>
        <el-table-column v-if="user.role === 1" label="操作" width="180">
          <template #default="scope">
            <el-button size="small" @click="handleEdit(scope.row)">编辑</el-button>
            <el-popconfirm title="确认删除?" @confirm="handleDelete(scope.row)">
              <template #reference>
                <el-button type="danger" size="small">删除</el-button>
              </template>
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

    <el-dialog v-model="dialogVisible" title="修改借阅记录" width="520px">
      <el-form :model="form" label-width="100px">
        <el-form-item label="借阅时间"><el-date-picker v-model="form.lendTime" type="datetime" value-format="YYYY-MM-DD HH:mm:ss" /></el-form-item>
        <el-form-item label="归还时间"><el-date-picker v-model="form.returnTime" type="datetime" value-format="YYYY-MM-DD HH:mm:ss" /></el-form-item>
        <el-form-item label="是否归还">
          <el-radio-group v-model="form.status">
            <el-radio label="0">未归还</el-radio>
            <el-radio label="1">已归还</el-radio>
          </el-radio-group>
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
  name: 'LendRecord',
  created() {
    this.load()
    const userStr = sessionStorage.getItem('user') || '{}'
    this.user = JSON.parse(userStr)
  },
  data() {
    return {
      form: {},
      search1: '',
      search2: '',
      search3: '',
      total: 0,
      currentPage: 1,
      pageSize: 10,
      tableData: [],
      user: {},
      dialogVisible: false,
      forms: [],
      loading: false,
      overdueCount: 0,
      returnedCount: 0
    }
  },
  methods: {
    handleSelectionChange(val) { this.forms = val },
    deleteBatch() {
      if (!this.forms.length) return ElMessage.warning('请选择数据！')
      request.post('/LendRecord/deleteRecords', this.forms).then(res => {
        if (res.code === '0' || res.code === 0) {
          ElMessage.success('批量删除成功')
          this.load()
        } else ElMessage.error(res.msg)
      })
    },
    load() {
      this.loading = true
      request.get('/LendRecord', {
        params: { pageNum: this.currentPage, pageSize: this.pageSize, search1: this.search1, search2: this.search2, search3: this.search3 }
      }).then(res => {
        this.tableData = res.data.records || []
        this.total = res.data.total || 0
        this.refreshCounters()
      }).finally(() => { this.loading = false })
    },
    refreshCounters() {
      this.overdueCount = this.tableData.filter(row => this.isOverdue(row)).length
      this.returnedCount = this.tableData.filter(row => row.status == 1).length
    },
    save() {
      const api = this.form.isbn ? request.put('/LendRecord/' + this.form.isbn, this.form) : request.post('/LendRecord' + this.form.isbn, this.form)
      api.then(res => {
        if (res.code == 0) ElMessage.success(this.form.isbn ? '更新成功' : '新增成功')
        else ElMessage.error(res.msg)
        this.load()
        this.dialogVisible = false
      })
    },
    clear() { this.search1 = ''; this.search2 = ''; this.search3 = ''; this.load() },
    handleEdit(row) { this.form = JSON.parse(JSON.stringify(row)); this.dialogVisible = true },
    handleSizeChange(pageSize) { this.pageSize = pageSize; this.load() },
    handleCurrentChange(pageNum) { this.currentPage = pageNum; this.load() },
    handleDelete(row) {
      request.post('LendRecord/deleteRecord', JSON.parse(JSON.stringify(row))).then(res => {
        if (res.code == 0) ElMessage.success('删除成功')
        else ElMessage.error(res.msg)
        this.load()
      })
    },
    isOverdue(row) {
      if (!row.deadtime) return false
      if (row.status == 1) return false
      return new Date(row.deadtime) < new Date()
    },
    overdueDays(row) {
      if (!row.deadtime) return 0
      const diff = new Date() - new Date(row.deadtime)
      return Math.max(0, Math.ceil(diff / (1000 * 60 * 60 * 24)))
    },
    calculateFine(row) {
      return this.overdueDays(row) * 1
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
.filter-card,.table-card { border-radius: 16px; margin-bottom: 16px; }
.table-actions { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
.pagination-wrap { display: flex; justify-content: flex-end; margin-top: 16px; }
@media (max-width: 768px) { .page-header { flex-direction: column; } }
</style>