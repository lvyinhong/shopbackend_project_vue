<template>
  <div>
    <!--面包屑导航区域-->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图区域-->
    <el-card class="box-card">
      <div
        id="main"
        style="height:400px"
      ></div>
    </el-card>
  </div>
</template>
<script>
import echarts from 'echarts'
import _ from 'loadsh'
export default {
  data() {
    return {
      options: {
        title: {
          text: '第一个 ECharts 实例'
        },
        tooltip: {},
        legend: {
          data: ['销量']
        },
        xAxis: {
          data: ['衬衫', '羊毛衫', '雪纺衫', '裤子', '高跟鞋', '袜子']
        },
        yAxis: {},
        series: [{
          name: '销量',
          type: 'bar',
          data: [5, 20, 36, 10, 10, 20]
        }]
      }
    }
  },
  created() {
  },
  async mounted() {
    // echarts 需要页面上的元素被渲染完毕时初始化图表
    var myChart = echarts.init(document.getElementById('main'))
    const { data: res } = await this.$http.get(`reports/type/1`)
    if (res.meta.status !== 200) return this.$message.error('获取折线图失败!')
    console.log(res)
    const result = _.merge(res.data, this.options)
    // 准备数据项
    myChart.setOption(result)
  },
  methods: {

  }
}
</script>
<style scoped lang="less">
</style>
