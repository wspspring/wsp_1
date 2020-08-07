<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>参数列表</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图区域 -->
        <el-card>
            <!-- 警告区域 -->
            <el-alert show-icon title="注意：只允许为第三级目录设置相关参数！" type="warning" :closable="false"></el-alert>
            <!-- 选择商品分类区域 -->
            <el-row class="cat_opt">
                <el-col>
                    <span>选择商品分类：</span>
                    <el-cascader v-model="selectedCateKeys" :options="cateList" :props="cateProps" @change="handleChange"></el-cascader>
                </el-col>
            </el-row>
            <!-- tab页签区域 -->
            <el-tabs v-model="activeName" @tab-click="handleTabClick">
                <!-- 添加动态参数的面板 -->
                <el-tab-pane label="动态参数" name="many">
                    <el-button type="primary" size="mini" :disabled="isBtnDiabled" @click="addDialogVisiable = true">添加参数</el-button>
                    <!-- 动态参数表格 -->
                    <el-table :data="manyTableData" border stripe>
                        <el-table-column type="expand">
                            <template slot-scope="scope">
                                <el-tag v-for="(item, i) in scope.row.attr_vals" :key="i" closable @close="handleClose(i,scope.row)">{{item}}</el-tag>
                                <!-- 输入的文本框 -->
                                <el-input
                                class="input-new-tag inputTag"
                                v-if="scope.row.inputVisible"
                                v-model="scope.row.inputValue"
                                ref="saveTagInput"
                                size="small"
                                @keyup.enter.native="handleInputConfirm(scope.row)"
                                @blur="handleInputConfirm(scope.row)"
                                >
                                </el-input>
                                <!-- 添加按钮 -->
                                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
                            </template>
                        </el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column prop="attr_name" label="参数名称"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="">
                                <el-button size="mini" type="primary" icon="el-icon-edit">搜索</el-button>
                                <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
                <!-- 添加静态属性的面板 -->
                <el-tab-pane label="静态属性" name="only">
                    <el-button type="primary" size="mini" :disabled="isBtnDiabled" @click="addDialogVisiable = true">添加属性</el-button>
                    <!-- 静态属性表格 -->
                    <el-table :data="onlyTableData" border stripe>
                        <el-table-column type="expand"></el-table-column>
                        <el-table-column type="index" label="#"></el-table-column>
                        <el-table-column prop="attr_name" label="参数名称"></el-table-column>
                        <el-table-column label="操作">
                            <template slot-scope="">
                                <el-button size="mini" type="primary" icon="el-icon-edit">搜索</el-button>
                                <el-button size="mini" type="danger" icon="el-icon-delete">删除</el-button>
                            </template>
                        </el-table-column>
                    </el-table>
                </el-tab-pane>
            </el-tabs>
        </el-card>
        <el-dialog
        :title="'添加' + titleText"
        :visible.sync="addDialogVisiable"
        width="50%" @close="addDialogClosed">
        <!-- 添加参数的对话框 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
            <el-form-item :label="titleText" prop="attr_name">
                <el-input v-model="addForm.attr_name"></el-input>
            </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
            <el-button @click="addDialogVisiable = false">取 消</el-button>
            <el-button type="primary" @click="addParams">确 定</el-button>
        </span>
        </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      cateList: [],
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children',
        expandTrigger: 'hover'
      },
      // 级连选择框的选择数组
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 动态表格数据
      manyTableData: [],
      // 静态表格数据
      onlyTableData: [],
      // 控制添加对话框的显示与隐藏式
      addDialogVisiable: false,
      // 添加参数的表单
      addForm: {
        attr_name: ''
      },
      addFormRules: {
        attr_name: [
          { required: true, trigger: 'blur', message: '请输入参数' }
        ]
      },
      // 控制文本框的显示与隐藏
      inputVisible: false,
      inputValue: ''
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    // 获取所有的商品分类列表
    async getCateList () {
      const { data: res } = await this.$http.get('categories', { params: { type: 3 } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取失败')
      }
      this.cateList = res.data
      console.log(this.cateList)
    },
    // 级连选择框变化监听
    async handleChange () {
      console.log(this.selectedCateKeys)
      this.getParamsData()
    },
    // tab页签点击事件
    handleTabClick () {
      console.log(this.activeName)
      this.getParamsData()
    },
    async getParamsData () {
      if (this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        this.manyTableData = []
        this.onlyTableData = []
        return this.selectedCateKeys
      }
      // 根据所选分类获取的id，和当前选中的面板，获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数列表失败')
      }
      console.log(res.data)
      // 将属性值按照空格分割组合为数组
      res.data.forEach(item => {
        item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : []
        // 控制文本框的显示与隐藏
        item.inputVisible = false
        // 文本框的输入值
        item.inputValue = ''
      })
      if (this.activeName === 'many') {
        this.manyTableData = res.data
      } else {
        this.onlyTableData = res.data
      }
    },
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addParams () {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('认证失败')
        }
        console.log(this.addForm)
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`,
          {
            attr_name: this.addForm.attr_name,
            attr_sel: this.activeName
          }
        )
        if (res.meta.status !== 201) {
          return this.$message.error('添加失败')
        }
        this.$message.success('添加成功')
        this.addDialogVisiable = false
        this.getParamsData()
      })
    },
    // 文本框失去了焦点或摁下enter均会触发
    async handleInputConfirm (row) {
      console.log(row)
      if (row.inputValue.trim().length === 0) {
        row.inputValue = ''
        row.inputVisible = false
        return 0
      }
      // 用户输入的内容正确需要做后续的处理
      row.attr_vals.push(row.inputValue.trim())
      row.inputValue = ''
      row.inputVisible = false
      // 发起请求
      this.saveAttrValues(row)
    },
    // 保存属性参数
    async saveAttrValues (row) {
      const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
        {
          attr_vals: row.attr_vals.join(' '),
          attr_name: row.attr_name,
          attr_sel: row.attr_sel
        })
      if (res.meta.status !== 200) {
        return this.$message.error('失败')
      }
      this.$message.success('成功')
    },
    // 点击按钮触发
    showInput (row) {
      row.inputVisible = true
      // 让文本框自动获取焦点 $nextTick当页面看上元素被重新渲染后才执行
      this.$nextTick(_ => {
        this.$refs.saveTagInput.$refs.input.focus()
      })
    },
    // 删除对应的参数可选项
    handleClose (i, row) {
      row.attr_vals.splice(i, 1)
      this.saveAttrValues(row)
    }
  },
  computed: {
    // 如果按钮需要被禁用，返回true，否则返回false
    isBtnDiabled () {
      if (this.selectedCateKeys.length !== 3) {
        return true
      } else {
        return false
      }
    },
    // 当前选中的三级分类的id
    cateId () {
      if (this.selectedCateKeys.length === 3) {
        console.log(this.selectedCateKeys[2])
        return this.selectedCateKeys[2]
      } else {
        return null
      }
    },
    titleText () {
      if (this.activeName === 'many') {
        return '动态参数'
      } else {
        return '静态属性'
      }
    }
  }
}
</script>

<style lang='less'>
// 无法使用scoped 否则无法设置级连选择器的样式
.cat_opt {
    margin: 15px 0px;
}
div.el-popper.el-cascader__dropdown {
     top: 200px !important;
}
.el-cascader-panel {
    height: 300px;
}
.el-tag {
    margin: 5px 10px;
}
.input-new-tag.inputTag {
    width: 120px;
}
</style>
