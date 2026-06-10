<template>
  <div class="page-shell">
    <div class="page-header">
      <div>
        <div class="page-eyebrow">Book Management</div>
        <h2>图书管理</h2>
        <p>支持搜索、分页、借阅、归还和批量删除，是数据库实践中最能体现业务流程的核心页面。</p>
      </div>
      <div class="header-actions" v-if="user.role == 1">
        <el-button type="primary" @click="add">上架图书</el-button>
        <el-popconfirm title="确认删除所选图书？" @confirm="deleteBatch">
          <template #reference>
            <el-button type="danger">批量删除</el-button>
          </template>
        </el-popconfirm>
      </div>
    </div>

    <el-card shadow="never" class="search-card">
      <el-form inline size="small" class="search-form">
        <el-form-item label="图书编号">
          <el-input v-model="search1" placeholder="请输入图书编号" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="图书名称">
          <el-input v-model="search2" placeholder="请输入图书名称" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="作者">
          <el-input v-model="search3" placeholder="请输入作者" clearable>
            <template #prefix><el-icon><Search /></el-icon></template>
          </el-input>
        </el-form-item>
        <el-form-item label="分类">
          <el-select v-model="searchCategory" placeholder="请选择分类" clearable style="width: 160px">
            <el-option label="全部分类" value="" />
            <el-option label="数据库" value="数据库" />
            <el-option label="编程" value="编程" />
            <el-option label="文学" value="文学" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button type="primary" @click="load">查询</el-button>
        </el-form-item>
        <el-form-item>
          <el-button @click="clear">重置</el-button>
        </el-form-item>
        <el-form-item class="notice-item" v-if="numOfOutDataBook != 0">
          <el-popconfirm confirm-button-text="查看" cancel-button-text="取消" icon-color="red" title="您有图书已逾期，请尽快归还" @confirm="toLook">
            <template #reference>
              <el-button type="warning">逾期通知</el-button>
            </template>
          </el-popconfirm>
        </el-form-item>
      </el-form>
    </el-card>

    <el-card shadow="never" class="table-card">
      <el-table :data="tableData" stripe border v-loading="loading" @selection-change="handleSelectionChange">
        <el-table-column v-if="user.role == 1" type="selection" width="55" />
        <el-table-column prop="isbn" label="图书编号" sortable min-width="140" />
        <el-table-column prop="name" label="图书名称" min-width="150" />
        <el-table-column prop="category" label="分类" min-width="120">
          <template #default="scope">
            <el-tag type="info">{{ scope.row.category || '未分类' }}</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="price" label="价格" sortable min-width="100" />
        <el-table-column prop="author" label="作者" min-width="120" />
        <el-table-column prop="publisher" label="出版社" min-width="150" />
        <el-table-column prop="createTime" label="出版时间" sortable min-width="130" />
        <el-table-column prop="status" label="状态" width="110">
          <template #default="scope">
            <el-tag v-if="scope.row.status == 0" type="warning">已借阅</el-tag>
            <el-tag v-else type="success">未借阅</el-tag>
          </template>
        </el-table-column>
        <el-table-column fixed="right" label="操作" min-width="220">
          <template #default="scope">
            <el-button size="small" @click="handleEdit(scope.row)" v-if="user.role == 1">修改</el-button>
            <el-popconfirm title="确认删除?" @confirm="handleDelete(scope.row.id)" v-if="user.role == 1">
              <template #reference>
                <el-button type="danger" size="small">删除</el-button>
              </template>
            </el-popconfirm>
            <el-button size="small" type="primary" @click="handlelend(scope.row.id, scope.row.isbn, scope.row.name, scope.row.borrownum)" v-if="user.role == 2" :disabled="scope.row.status == 0">借阅</el-button>
            <el-popconfirm title="确认还书?" @confirm="handlereturn(scope.row.id, scope.row.isbn, scope.row.borrownum)" v-if="user.role == 2">
              <template #reference>
                <el-button type="danger" size="small" :disabled="(this.isbnArray.indexOf(scope.row.isbn)) == -1 || scope.row.status == 1">还书</el-button>
              </template>
            </el-popconfirm>
          </template>
        </el-table-column>
      </el-table>

      <el-empty v-if="!loading && !tableData.length" description="暂无图书数据" />

      <div class="pagination-wrap">
        <el-pagination v-model:currentPage="currentPage" :page-sizes="[5, 10, 20]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total" @size-change="handleSizeChange" @current-change="handleCurrentChange" />
      </div>
    </el-card>

    <el-dialog v-model="dialogVisible3" v-if="numOfOutDataBook != 0" title="逾期详情" width="50%">
      <el-table :data="outDateBook" style="width: 100%">
        <el-table-column prop="isbn" label="图书编号" />
        <el-table-column prop="bookName" label="书名" />
        <el-table-column prop="lendtime" label="借阅日期" />
        <el-table-column prop="deadtime" label="截至日期" />
      </el-table>
      <template #footer>
        <el-button type="primary" @click="dialogVisible3 = false">确认</el-button>
      </template>
    </el-dialog>

    <el-dialog v-model="dialogVisible" title="上架书籍" width="34%">
      <el-form :model="form" label-width="100px">
        <el-form-item label="图书编号"><el-input v-model="form.isbn" /></el-form-item>
        <el-form-item label="图书名称"><el-input v-model="form.name" /></el-form-item>
        <el-form-item label="分类">
          <el-select v-model="form.category" style="width: 100%" placeholder="请选择分类">
            <el-option label="数据库" value="数据库" />
            <el-option label="编程" value="编程" />
            <el-option label="文学" value="文学" />
          </el-select>
        </el-form-item>
        <el-form-item label="价格"><el-input v-model="form.price" /></el-form-item>
        <el-form-item label="作者"><el-input v-model="form.author" /></el-form-item>
        <el-form-item label="出版社"><el-input v-model="form.publisher" /></el-form-item>
        <el-form-item label="出版时间"><el-date-picker v-model="form.createTime" value-format="YYYY-MM-DD" type="date" clearable style="width: 100%" /></el-form-item>
      </el-form>
      <template #footer>
        <el-button @click="dialogVisible = false">取消</el-button>
        <el-button type="primary" @click="save">确定</el-button>
      </template>
    </el-dialog>

    <el-dialog v-model="dialogVisible2" title="修改书籍信息" width="34%">
      <el-form :model="form" label-width="100px">
        <el-form-item label="图书编号"><el-input v-model="form.isbn" /></el-form-item>
        <el-form-item label="图书名称"><el-input v-model="form.name" /></el-form-item>
        <el-form-item label="分类">
          <el-select v-model="form.category" style="width: 100%" placeholder="请选择分类">
            <el-option label="数据库" value="数据库" />
            <el-option label="编程" value="编程" />
            <el-option label="文学" value="文学" />
          </el-select>
        </el-form-item>
        <el-form-item label="价格"><el-input v-model="form.price" /></el-form-item>
        <el-form-item label="作者"><el-input v-model="form.author" /></el-form-item>
        <el-form-item label="出版社"><el-input v-model="form.publisher" /></el-form-item>
        <el-form-item label="出版时间"><el-date-picker v-model="form.createTime" value-format="YYYY-MM-DD" type="date" clearable style="width: 100%" /></el-form-item>
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
  created() {
    const userStr = sessionStorage.getItem('user') || '{}'
    this.user = JSON.parse(userStr)
    this.load()
  },
  name: 'Book',
  methods: {
    handleSelectionChange(val) { this.ids = val.map(v => v.id) },
    deleteBatch() {
      if (!this.ids.length) return ElMessage.warning('请选择数据！')
      request.post('/book/deleteBatch', this.ids).then(res => {
        if (res.code === '0') { ElMessage.success('批量删除成功'); this.load() } else ElMessage.error(res.msg)
      })
    },
    load() {
      this.loading = true
      this.numOfOutDataBook = 0
      this.outDateBook = []
      request.get('/book', { params: { pageNum: this.currentPage, pageSize: this.pageSize, search1: this.search1, search2: this.search2, search3: this.search3, searchCategory: this.searchCategory } }).then(res => {
        this.tableData = res.data.records || []
        this.total = res.data.total || 0
      }).finally(() => { this.loading = false })

      if (this.user.role == 2) {
        request.get('/bookwithuser', { params: { pageNum: '1', pageSize: this.total, search1: '', search2: '', search3: this.user.id } }).then(res => {
          this.bookData = res.data.records || []
          this.number = this.bookData.length
          const nowDate = new Date()
          this.isbnArray = []
          for (let i = 0; i < this.number; i++) {
            this.isbnArray[i] = this.bookData[i].isbn
            const dDate = new Date(this.bookData[i].deadtime)
            if (dDate < nowDate) {
              this.outDateBook[this.numOfOutDataBook] = { isbn: this.bookData[i].isbn, bookName: this.bookData[i].bookName, deadtime: this.bookData[i].deadtime, lendtime: this.bookData[i].lendtime }
              this.numOfOutDataBook++
            }
          }
        })
      }
    },
    clear() { this.search1 = ''; this.search2 = ''; this.search3 = ''; this.searchCategory = ''; this.load() },
    handleDelete(id) { request.delete('book/' + id).then(res => { if (res.code == 0) ElMessage.success('删除成功'); else ElMessage.error(res.msg); this.load() }) },
    handlereturn(id, isbn, bn) {
      this.form.status = '1'
      this.form.id = id
      request.put('/book', this.form).then(res => {
        if (res.code == 0) ElMessage.success('还书成功'); else ElMessage.error(res.msg)
        this.form3.isbn = isbn
        this.form3.readerId = this.user.id
        const endDate = moment(new Date()).format('yyyy-MM-DD HH:mm:ss')
        this.form3.returnTime = endDate
        this.form3.status = '1'
        this.form3.borrownum = bn
        request.put('/LendRecord1/', this.form3).then(() => {
          const form3 = { isbn, bookName: this.form.name, nickName: this.user.username, id: this.user.id, lendtime: endDate, deadtime: endDate, prolong: 1 }
          request.post('/bookwithuser/deleteRecord', form3).then(() => { this.load() })
        })
      })
    },
    handlelend(id, isbn, name, bn) {
      if (this.number == 5) return ElMessage.warning('您不能再借阅更多的书籍了')
      if (this.numOfOutDataBook != 0) return ElMessage.warning('在您归还逾期书籍前不能再借阅书籍')
      this.form.status = '0'
      this.form.id = id
      this.form.borrownum = bn + 1
      request.put('/book', this.form).then(res => { if (res.code == 0) ElMessage.success('借阅成功'); else ElMessage.error(res.msg) })
      this.form2.status = '0'
      this.form2.isbn = isbn
      this.form2.bookname = name
      this.form2.readerId = this.user.id
      this.form2.borrownum = bn + 1
      const startDate = moment(new Date()).format('yyyy-MM-DD HH:mm:ss')
      this.form2.lendTime = startDate
      request.post('/LendRecord', this.form2).then(() => { this.load() })
      const form3 = { isbn, bookName: name, nickName: this.user.username, id: this.user.id, lendtime: startDate }
      const nowDate = new Date(startDate)
      nowDate.setDate(nowDate.getDate() + 30)
      form3.deadtime = moment(nowDate).format('yyyy-MM-DD HH:mm:ss')
      form3.prolong = 1
      request.post('/bookwithuser/insertNew', form3).then(() => { this.load() })
    },
    add() { this.dialogVisible = true; this.form = {} },
    save() {
      if (this.form.id) {
        request.put('/book', this.form).then(res => { if (res.code == 0) ElMessage.success('修改书籍信息成功'); else ElMessage.error(res.msg); this.load(); this.dialogVisible2 = false })
      } else {
        this.form.borrownum = 0
        this.form.status = 1
        request.post('/book', this.form).then(res => { if (res.code == 0) ElMessage.success('上架书籍成功'); else ElMessage.error(res.msg); this.load(); this.dialogVisible = false })
      }
    },
    handleEdit(row) { this.form = JSON.parse(JSON.stringify(row)); this.dialogVisible2 = true },
    handleSizeChange(pageSize) { this.pageSize = pageSize; this.load() },
    handleCurrentChange(pageNum) { this.currentPage = pageNum; this.load() },
    toLook() { this.dialogVisible3 = true }
  },
  data() {
    return {
      loading: false,
      form: {},
      form2: {},
      form3: {},
      dialogVisible: false,
      dialogVisible2: false,
      search1: '',
      search2: '',
      search3: '',
      searchCategory: '',
      total: 10,
      currentPage: 1,
      pageSize: 10,
      tableData: [],
      user: {},
      number: 0,
      bookData: [],
      isbnArray: [],
      outDateBook: [],
      numOfOutDataBook: 0,
      dialogVisible3: true,
      ids: []
    }
  }
}
</script>

<style scoped>
.page-shell { padding: 18px; }
.page-header { display: flex; justify-content: space-between; gap: 16px; align-items: flex-start; margin-bottom: 16px; }
.page-eyebrow { color: #2563eb; font-size: 12px; font-weight: 700; letter-spacing: 0.08em; text-transform: uppercase; }
.page-header h2 { margin: 8px 0 6px; font-size: 26px; }
.page-header p { margin: 0; color: #64748b; }
.header-actions { display: flex; gap: 10px; flex-wrap: wrap; }
.search-card,.table-card { border-radius: 16px; margin-bottom: 16px; }
.search-form { margin-bottom: -8px; }
.notice-item { margin-left: auto; }
.pagination-wrap { display: flex; justify-content: flex-end; padding-top: 16px; }
@media (max-width: 768px) { .page-header { flex-direction: column; } .notice-item { margin-left: 0; } }
</style>