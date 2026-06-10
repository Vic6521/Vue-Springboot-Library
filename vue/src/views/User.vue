<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <h2>读者管理</h2>
        <p>支持多条件查询、批量删除和编辑维护，体现读者信息的完整数据库操作流程。</p>
      </div>
    </div>

    <el-card class="filter-card" shadow="never">
      <el-form :inline="true" size="small" class="filter-form">
        <el-form-item label="读者编号">
          <el-input v-model="search1" placeholder="请输入读者编号" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="姓名">
          <el-input v-model="search2" placeholder="请输入姓名" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="电话号码">
          <el-input v-model="search3" placeholder="请输入电话号码" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="地址">
          <el-input v-model="search4" placeholder="请输入地址" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
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
        <el-tag type="info">共 {{ total }} 条读者数据</el-tag>
      </div>

      <el-table :data="tableData" stripe border v-loading="loading" @selection-change="handleSelectionChange">
        <el-table-column v-if="user.role == 1" type="selection" width="55" />
        <el-table-column prop="id" label="读者编号" sortable />
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="nickName" label="姓名" />
        <el-table-column prop="phone" label="电话号码" />
        <el-table-column prop="sex" label="性别">
          <template #default="scope">
            <el-tag :type="scope.row.sex === '男' ? 'primary' : 'success'">{{ scope.row.sex }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="address" label="地址" show-overflow-tooltip />
        <el-table-column fixed="right" label="操作" width="170">
          <template #default="scope">
            <el-button size="small" @click="handleEdit(scope.row)">编辑</el-button>
            <el-popconfirm title="确认删除?" @confirm="handleDelete(scope.row.id)">
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

    <el-dialog v-model="dialogVisible" title="编辑读者信息" width="520px">
      <el-form :model="form" label-width="100px">
        <el-form-item label="用户名"><el-input v-model="form.username" /></el-form-item>
        <el-form-item label="昵称"><el-input v-model="form.nickName" /></el-form-item>
        <el-form-item label="电话号码"><el-input v-model="form.phone" /></el-form-item>
        <el-form-item label="性别">
          <el-radio-group v-model="form.sex">
            <el-radio label="男">男</el-radio>
            <el-radio label="女">女</el-radio>
          </el-radio-group>
        </el-form-item>
        <el-form-item label="地址"><el-input v-model="form.address" type="textarea" :rows="3" /></el-form-item>
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
  name: 'User',
  created() {
    this.load()
    const userStr = sessionStorage.getItem('user') || '{}'
    this.user = JSON.parse(userStr)
  },
  data() {
    return {
      form: {},
      dialogVisible: false,
      search1: '',
      search2: '',
      search3: '',
      search4: '',
      total: 0,
      currentPage: 1,
      pageSize: 10,
      tableData: [],
      user: {},
      ids: [],
      loading: false
    }
  },
  methods: {
    handleSelectionChange(val) {
      this.ids = val.map(v => v.id)
    },
    deleteBatch() {
      if (!this.ids.length) return ElMessage.warning('请选择数据！')
      request.post('/user/deleteBatch', this.ids).then(res => {
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
      request.get('user/usersearch', {
        params: {
          pageNum: this.currentPage,
          pageSize: this.pageSize,
          search1: this.search1,
          search2: this.search2,
          search3: this.search3,
          search4: this.search4
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
      this.search4 = ''
      this.load()
    },
    handleDelete(id) {
      request.delete('user/' + id).then(res => {
        if (res.code == 0) ElMessage.success('删除成功')
        else ElMessage.error(res.msg)
        this.load()
      })
    },
    save() {
      const api = this.form.id ? request.put('/user', this.form) : request.post('/user', this.form)
      api.then(res => {
        if (res.code == 0) ElMessage.success(this.form.id ? '更新成功' : '添加成功')
        else ElMessage.error(res.msg)
        this.load()
        this.dialogVisible = false
      })
    },
    handleEdit(row) {
      this.form = JSON.parse(JSON.stringify(row))
      this.dialogVisible = true
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