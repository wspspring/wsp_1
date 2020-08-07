<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>数据统计</el-breadcrumb-item>
        <el-breadcrumb-item>数据报表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <div id="main" style="width:600px;height:400px"></div>
        </el-card>
    </div>
</template>
<script>
// import echarts from 'echarts'
export default {
  data () {
    return {}
  },
  created () {},
  // 此时页面上的元素已经加载完毕
  //   async mounted () {
  //     // 基于创建好的dom，实例化echarts
  //     var myCharts = echarts.init(document.getElementById('main'))
  //     const { data: res } = await this.$http.get('reports/type/1')
  //     if (res.meta.status !== 200) {
  //       return this.$message.error('获取失败')
  //     }
  //   },
  methods: {}
}
</script>
<style scoped>

</style>
