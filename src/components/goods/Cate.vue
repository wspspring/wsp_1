<template>
    <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card>
        <el-row>
            <el-button type="primary" @click="showAddCatDialog">添加分类</el-button>
        </el-row>
        <!-- 表格区域 -->
        <table-tree class="treeTable" :data="cateList" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text="#" border :show-row-hover="false">
            <!-- 商品是否有效的作用域插槽 slot匹配插槽名称 slot-scope获取值 -->
            <template slot="isOk" slot-scope="scope">
                <i class="el-icon-success" v-if="scope.row.at_delete === false" style="color:lightgreen;"></i>
                <i class="el-icon-error" v-else style="color:red;"></i>
            </template>
            <template slot="order" slot-scope="scope">
                <el-tag size="mini" v-if="scope.row.cat_level===0">一级</el-tag>
                <el-tag type="success" size="mini" v-else-if="scope.row.cat_level===1">二级</el-tag>
                <el-tag type="warning" size="mini" v-else>三级</el-tag>
            </template>
            <template slot="opt" slot-scope="">
                <el-button type="primary" size="mini" icon="el-icon-edit">编辑</el-button>
                <el-button type="danger" size="mini" icon="el-icon-delete">删除</el-button>
            </template>
        </table-tree>
        <!-- 分页区域 -->
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[2, 5, 8]"
            :page-size="queryInfo.pagesize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog
    title="添加分类"
    :visible.sync="addCatDialogVisiable"
    width="50%">
    <el-form :model="addCatForm" :rules="addCatFormRules" ref="addCatFormRef" label-width="100px">
        <el-form-item label="分类名称：" prop="cat_name">
            <el-input v-model="addCatForm.name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
            <!-- options用来指定数据源 props用来指定配置对象 -->
            <el-cascader
            expand-trigger="hover"
            :props="cascaderProps"
            v-model="selectedKeys"
            :options="parentCatList"
            @change="parentCateChanged">
            </el-cascader>
        </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
        <el-button @click="addCatDialogVisiable = false">取消</el-button>
        <el-button type="primary" @click="addCatDialogVisiable = false">确认</el-button>
    </span>
    </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      queryInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      total: 0,
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name'
        },
        {
          label: '是否有效',
          // 表示当前列定义为模版
          type: 'template',
          // 模版名称
          template: 'isOk'
        },
        {
          label: '排序',
          type: 'template',
          template: 'order'
        },
        {
          label: '操作',
          type: 'template',
          template: 'opt'
        }
      ],
      addCatDialogVisiable: false,
      addCatForm: {
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        cat_level: 0
      },
      // 添加分类的验证规则
      addCatFormRules: {
        cat_name: [
          { required: true, trigger: 'blur', message: '请输入商品分类名称' }
        ]
      },
      parentCatList: [],
      // 指定级连选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 选中的父级分类的id数组
      selectedKeys: []
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: this.queryInfo })
      if (res.meta.status !== 200) {
        return this.$message.error('获取失败')
      }
      // 商品列表
      this.cateList = res.data.result
      console.log(this.cateList)
      // 商品总数
      this.total = res.data.total
    },
    // 监听pagenum变化
    handleSizeChange (newSize) {
      this.queryInfo.pagesize = newSize
      this.getCateList()
    },
    handleCurrentChange (newPage) {
      this.queryInfo.pagenum = newPage
      this.getCateList()
    },
    showAddCatDialog () {
      this.getParentCatList()
      this.addCatDialogVisiable = true
    },
    async getParentCatList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 2 } })
      if (res.meta.status !== 200) {
        return this.$message.error('失败')
      }
      this.parentCatList = res.data
      console.log(this.parentCatList)
    },
    // 问题：第二级不显示
    // getTreeData (data) {
    //   // 循环遍历json数据
    // //   console.log(data)
    //   for (var i = 0; i < data.length; i++) {
    //     // console.log(data[i])
    //     if (data[i].children === undefined) {
    //       // children若为空数组，则将children设为undefined
    //       data[i].children = undefined
    //     } else {
    //       // children若不为空数组，则继续 递归调用 本方法
    //       this.getTreeData(data[i].children)
    //     }
    //   }
    //   return data
    // },
    // 选择项发生变化时触发这个函数
    parentCateChanged () {
      console.log(this.selectedKeys)
    }
  }
}
</script>
<style lang='less' scoped>
.treeTable {
    margin-top: 15px;
}
.el-cascader {
    width: 100%;
}
</style>
