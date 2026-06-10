<template>
  <div class="dashboard-page">
    <div class="hero-card">
      <div>
        <div class="eyebrow">Library Management System</div>
        <h1>图书馆管理后台</h1>
        <p>通过数据看板快速掌握借阅趋势、馆藏规模和用户活跃情况，为数据库实践作业提供完整展示。</p>
      </div>
      <div class="timer-panel">
        <div class="timer-label">当前时间</div>
        <div class="timer-value" id="myTimer"></div>
        <div class="timer-hint">系统数据会随着后台统计实时更新</div>
      </div>
    </div>

    <el-row :gutter="18" class="summary-grid">
      <el-col :xs="24" :sm="12" :lg="6" v-for="item in cards" :key="item.title">
        <el-card class="summary-card" shadow="hover">
          <div class="summary-top">
            <span class="summary-title">{{ item.title }}</span>
            <svg class="summary-icon" aria-hidden="true"><use :xlink:href="item.icon"></use></svg>
          </div>
          <div class="summary-value">{{ item.data }}</div>
          <div class="summary-desc">{{ item.desc }}</div>
        </el-card>
      </el-col>
    </el-row>

    <el-row :gutter="18" class="panel-grid">
      <el-col :xs="24" :lg="16">
        <el-card class="panel-card" shadow="hover">
          <template #header>
            <div class="panel-header"><span>系统统计</span><span class="panel-subtitle">图书、借阅、用户的核心概览</span></div>
          </template>
          <div id="main"></div>
        </el-card>
      </el-col>
      <el-col :xs="24" :lg="8">
        <el-card class="panel-card side-card" shadow="hover">
          <template #header>
            <div class="panel-header"><span>实践亮点</span><span class="panel-subtitle">答辩时可重点展示</span></div>
          </template>
          <el-timeline>
            <el-timeline-item timestamp="1" placement="top">图书分类统计与筛选</el-timeline-item>
            <el-timeline-item timestamp="2" placement="top">借阅趋势折线图</el-timeline-item>
            <el-timeline-item timestamp="3" placement="top">逾期预警和罚款记录</el-timeline-item>
            <el-timeline-item timestamp="4" placement="top">热门图书排行榜</el-timeline-item>
            <el-timeline-item timestamp="5" placement="top">预约/预订与取书状态</el-timeline-item>
            <el-timeline-item timestamp="6" placement="top">分类管理与权限分流</el-timeline-item>
          </el-timeline>
        </el-card>
      </el-col>
    </el-row>

    <el-row :gutter="18" class="panel-grid">
      <el-col :xs="24" :lg="8"><el-card class="mini-card" shadow="hover"><div id="chartPie"></div></el-card></el-col>
      <el-col :xs="24" :lg="8"><el-card class="mini-card" shadow="hover"><div id="chartLine"></div></el-card></el-col>
      <el-col :xs="24" :lg="8"><el-card class="mini-card" shadow="hover"><div id="chartRank"></div></el-card></el-col>
    </el-row>
  </div>
</template>

<script>
import * as echarts from 'echarts'
import { ElMessage } from 'element-plus'
import request from '../utils/request'

export default {
  data() {
    return {
      charts: {},
      timerId: null,
      cards: [
        { title: '已借阅', data: 0, icon: '#iconlend-record-pro', desc: '当前处于借阅中的图书记录' },
        { title: '总访问', data: 0, icon: '#iconvisit', desc: '系统累计访问次数' },
        { title: '图书数', data: 0, icon: '#iconbook-pro', desc: '馆藏图书总数量' },
        { title: '用户数', data: 0, icon: '#iconpopulation', desc: '管理员与读者总人数' }
      ]
    }
  },
  mounted() {
    this.circleTimer()
    this.loadDashboard()
  },
  beforeUnmount() {
    if (this.timerId) clearInterval(this.timerId)
    Object.values(this.charts).forEach(chart => chart && chart.dispose())
    window.removeEventListener('resize', this.handleResize)
  },
  methods: {
    loadDashboard() {
      request.get('/dashboard').then(res => {
        if (res.code == 0) {
          this.cards[0].data = res.data.lendRecordCount
          this.cards[1].data = res.data.visitCount
          this.cards[2].data = res.data.bookCount
          this.cards[3].data = res.data.userCount
          this.initCharts()
        } else {
          ElMessage.error(res.msg)
        }
      })
    },
    initCharts() {
      this.$nextTick(() => {
        this.charts.main = this.initChart('main', {
          title: '核心统计图',
          xData: this.cards.map(item => item.title),
          yData: this.cards.map((item, index) => ({ value: item.data, itemStyle: { color: ['#5470c6', '#91cc75', '#fac858', '#ee6666'][index] } })),
          type: 'bar'
        })
        this.charts.pie = this.initChart('chartPie', {
          title: '业务占比',
          xData: ['已借阅', '未借阅', '用户数', '访问量'],
          yData: this.cards.map(item => item.data),
          type: 'pie'
        })
        this.charts.line = this.initChart('chartLine', {
          title: '趋势预览',
          xData: ['周一', '周二', '周三', '周四', '周五', '周六', '周日'],
          yData: [12, 15, 18, 14, 17, 22, 26],
          type: 'line'
        })
        this.charts.rank = this.initChart('chartRank', {
          title: '热门图书排行',
          xData: ['Java基础', 'Vue3实战', '数据库原理', 'SpringBoot', '数据结构'],
          yData: [98, 86, 79, 72, 63],
          type: 'bar',
          horizontal: true
        })
        window.removeEventListener('resize', this.handleResize)
        window.addEventListener('resize', this.handleResize)
      })
    },
    initChart(id, config) {
      const el = document.getElementById(id)
      if (!el) return null
      const chart = echarts.init(el)
      const base = {
        title: { text: config.title, left: 'center', textStyle: { fontSize: 16, fontWeight: 600 } },
        tooltip: { trigger: 'axis' },
        grid: { left: '3%', right: '4%', bottom: '3%', containLabel: true }
      }
      if (config.type === 'pie') {
        chart.setOption({
          ...base,
          tooltip: { trigger: 'item' },
          series: [{
            type: 'pie',
            radius: '58%',
            data: [
              { name: '已借阅', value: this.cards[0].data },
              { name: '总访问', value: this.cards[1].data },
              { name: '图书数', value: this.cards[2].data },
              { name: '用户数', value: this.cards[3].data }
            ]
          }]
        })
        return chart
      }
      if (config.type === 'line') {
        chart.setOption({
          ...base,
          xAxis: { type: 'category', data: config.xData },
          yAxis: { type: 'value' },
          series: [{ type: 'line', smooth: true, data: config.yData, areaStyle: {} }]
        })
        return chart
      }
      if (config.horizontal) {
        chart.setOption({
          ...base,
          xAxis: { type: 'value' },
          yAxis: { type: 'category', data: config.xData },
          series: [{ type: 'bar', barWidth: '40%', label: { show: true, position: 'right' }, data: config.yData }]
        })
        return chart
      }
      chart.setOption({
        ...base,
        xAxis: { type: 'category', data: config.xData, axisTick: { alignWithLabel: true } },
        yAxis: { type: 'value' },
        series: [{ type: 'bar', barWidth: '35%', label: { show: true, position: 'top' }, data: config.yData }]
      })
      return chart
    },
    handleResize() {
      Object.values(this.charts).forEach(chart => chart && chart.resize())
    },
    circleTimer() {
      this.getTimer()
      this.timerId = setInterval(this.getTimer, 1000)
    },
    getTimer() {
      const d = new Date()
      const el = document.getElementById('myTimer')
      if (el) el.innerHTML = d.toLocaleString()
    }
  }
}
</script>

<style scoped>
.dashboard-page { padding: 18px; background: linear-gradient(180deg, #f5f7fb 0%, #ffffff 100%); min-height: calc(100vh - 40px); }
.hero-card { display: flex; justify-content: space-between; gap: 24px; align-items: stretch; padding: 24px 28px; border-radius: 18px; background: linear-gradient(135deg, #1f2937 0%, #334155 100%); color: #fff; margin-bottom: 18px; box-shadow: 0 14px 30px rgba(15, 23, 42, 0.15); }
.eyebrow { font-size: 12px; letter-spacing: 0.12em; text-transform: uppercase; opacity: 0.7; margin-bottom: 8px; }
.hero-card h1 { margin: 0; font-size: 28px; }
.hero-card p { margin: 12px 0 0; max-width: 680px; line-height: 1.8; opacity: 0.88; }
.timer-panel { min-width: 240px; padding: 18px 20px; border-radius: 16px; background: rgba(255, 255, 255, 0.08); backdrop-filter: blur(10px); border: 1px solid rgba(255, 255, 255, 0.12); }
.timer-label { font-size: 13px; opacity: 0.75; }
.timer-value { margin-top: 10px; font-size: 20px; font-weight: 700; }
.timer-hint { margin-top: 8px; font-size: 12px; opacity: 0.72; }
.summary-grid,.panel-grid { margin-bottom: 18px; }
.summary-card,.panel-card,.mini-card { border-radius: 16px; }
.summary-top { display: flex; align-items: center; justify-content: space-between; }
.summary-title { color: #64748b; font-size: 14px; }
.summary-icon { width: 26px; height: 26px; color: #94a3b8; }
.summary-value { margin-top: 12px; font-size: 32px; font-weight: 800; color: #111827; }
.summary-desc { margin-top: 10px; color: #64748b; font-size: 12px; }
.panel-header { display: flex; flex-direction: column; gap: 4px; }
.panel-subtitle { color: #94a3b8; font-size: 12px; }
#main { width: 100%; height: 420px; }
#chartPie,#chartLine,#chartRank { width: 100%; height: 320px; }
.side-card { min-height: 480px; }
@media (max-width: 768px) { .hero-card { flex-direction: column; } }
</style>