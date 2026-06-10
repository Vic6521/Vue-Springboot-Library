<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>图书分类管理</h2>
        <p>通过分类表体现数据库实体设计，支持分类维护与按分类筛选图书。</p>
      </div>
    </div>

    <el-card class="filter-card" shadow="never">
      <div class="table-actions">
        <el-button type="primary" @click="openDialog()">新增分类</el-button>
        <el-tag type="info">共 {{ total }} 条分类</el-tag>
      </div>
      <el-table :data="tableData" stripe border v-loading="loading">
        <el-table-column prop="id" label="ID" width="90" />
        <el-table-column prop="name" label="分类名称" />
        <el-table-column prop="description" label="分类说明" />
        <el-table-column prop="status" label="状态" width="120">
          <template #default="scope">
            <el-tag :type="scope.row.status == 1 ? 'success' : 'warning'">{{ scope.row.status == 1 ? '启用' : '停用' }}</el-tag>
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

    <el-dialog v-model="dialogVisible" title="分类信息" width="480px">
      <el-form :model="form" label-width="100px">
        <el-form-item label="分类名称"><el-input v-model="form.name" /></el-form-item>
        <el-form-item label="分类说明"><el-input v-model="form.description" type="textarea" :rows="3" /></el-form-item>
        <el-form-item label="状态">
          <el-radio-group v-model="form.status">
            <el-radio :label="1">启用</el-radio>
            <el-radio :label="0">停用</el-radio>
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
  name: 'Category',
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
      request.get('/category').then(res => {
        this.tableData = res.data || []
        this.total = this.tableData.length
      }).finally(() => {
        this.loading = false
      })
    },
    openDialog(row) {
      this.form = row ? JSON.parse(JSON.stringify(row)) : { status: 1 }
      this.dialogVisible = true
    },
    save() {
      const api = this.form.id ? request.put('/category', this.form) : request.post('/category', this.form)
      api.then(res => {
        if (res.code == 0) ElMessage.success(this.form.id ? '更新成功' : '新增成功')
        else ElMessage.error(res.msg)
        this.dialogVisible = false
        this.load()
      })
    },
    handleDelete(id) {
      request.delete('/category/' + id).then(res => {
        if (res.code == 0) ElMessage.success('删除成功')
        else ElMessage.error(res.msg)
        this.load()
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
.filter-card { border-radius: 16px; }
.table-actions { display: flex; justify-content: space-between; align-items: center; margin-bottom: 12px; }
</style>