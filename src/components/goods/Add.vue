<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>商品管理</el-breadcrumb-item>
        <el-breadcrumb-item>添加商品</el-breadcrumb-item>
        </el-breadcrumb>
        <!-- 卡片视图 -->
        <el-card>
            <!-- 提示信息 -->
            <el-alert title="添加商品信息" type="info" center show-icon :closable="false">
        </el-alert>
        <!-- 步骤条 -->
        <el-steps :space="200" :active="activeIndex - 0" finish-status="success" align-center>
            <el-step title="基本信息"></el-step>
            <el-step title="商品参数"></el-step>
            <el-step title="商品属性"></el-step>
            <el-step title="商品图片"></el-step>
            <el-step title="商品内容"></el-step>
            <el-step title="完成"></el-step>
        </el-steps>
        <!-- tabs栏 -->
        <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" label-position="top">
            <el-tabs v-model="activeIndex" tab-position="left" :before-leave="beforeTabLeave" @tab-click="tabClicked">
                <el-tab-pane label="基本信息" name="0">
                    <el-form-item label="商品名称" prop="goods_name">
                        <el-input v-model="addForm.goods_name"></el-input>
                    </el-form-item>
                    <el-form-item label="商品价格" prop="goods_price">
                        <el-input v-model="addForm.goods_price" type="number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品数量" prop="goods_number">
                        <el-input v-model="addForm.goods_number" type="number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品价格" prop="goods_weight">
                        <el-input v-model="addForm.goods_weight" type="number"></el-input>
                    </el-form-item>
                    <el-form-item label="商品分类" prop="goods_cat">
                        <el-cascader
                            :options="cateList"
                            expand-trigger="hover"
                            :props="cateProps"
                            v-model="addForm.goods_cat"
                            @change="handleChange"
                            >
                        </el-cascader>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品参数" name="1">
                    <!-- 渲染表单item项 -->
                    <el-form-item v-for="item in manyTableData" :key="item.attr_id" :label="item.attr_name">
                        <!-- 复选框 -->
                        <el-checkbox-group v-model="item.attr_vals">
                            <el-checkbox :label="cb" v-for="(cb, i) in item.attr_vals" :key="i" border></el-checkbox>
                        </el-checkbox-group>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品属性" name="2">
                    <el-form-item :label="item.attr_name" v-for="item in onlyTableData" :key="item.attr_id">
                        <el-input v-model="item.attr_name"></el-input>
                    </el-form-item>
                </el-tab-pane>
                <el-tab-pane label="商品图片" name="3">
                    <!-- action图片上传的后台api地址 -->
                    <el-upload
                    :action="uploadUrl"
                    :on-preview="handlePreview"
                    :on-remove="handleRemove"
                    list-type="picture"
                    :headers="headerObj"
                    :on-success="handleSuccess">
                        <el-button size="small" type="primary">点击上传</el-button>
                    </el-upload>
                </el-tab-pane>
                <el-tab-pane label="商品内容" name="4">
                    <!-- 富文本编辑器组件 -->
                    <quill-editor v-model="addForm.goods_introduce">
                    </quill-editor>
                    <!-- 添加商品按钮 -->
                    <el-button type="primary" class="btnAddForm" @click="add">添加商品</el-button>
                </el-tab-pane>
            </el-tabs>
        </el-form>
        </el-card>
        <!-- 图片预览 -->
        <el-dialog
        title="图片预览"
        :visible.sync="previewVisible"
        width="50%">
        <img :src="previewPath" alt="" class="previewImg">
        </el-dialog>
    </div>
</template>
<script>
export default {
  data () {
    return {
      activeIndex: '0',
      addForm: {
        goods_name: '',
        goods_number: 0,
        goods_price: 0,
        goods_weight: 0,
        goods_cat: [],
        // 图片的数组
        pics: [],
        // 商品描述
        goods_introduce: '',
        attrs: []
      },
      // 商品分类列表
      cateList: [],
      cateProps: {
        label: 'cat_name',
        value: 'cat_id',
        children: 'children'
      },
      addFormRules: {
        goods_name: [
          { required: true, trigger: 'blur', message: '请输入商品名称' }
        ],
        goods_price: [
          { required: true, trigger: 'blur', message: '请输入商品价格' }
        ],
        goods_number: [
          { required: true, trigger: 'blur', message: '请输入商品数量' }
        ],
        goods_weight: [
          { required: true, trigger: 'blur', message: '请输入商品重量' }
        ],
        goods_cat: [
          { required: true, trigger: 'blur', message: '请选择商品类别' }
        ]
      },
      // 动态参数列表数据
      manyTableData: [],
      onlyTableData: [],
      uploadUrl: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 上传图片的headers设置
      headerObj: {
        Authorization: window.sessionStorage.getItem('token')
      },
      previewPath: '',
      previewVisible: false
    }
  },
  created () {
    this.getCateList()
  },
  methods: {
    async getCateList () {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品类别失败')
      }
      this.cateList = res.data
      console.log(this.cateList)
    },
    handleChange () {
      console.log(this.addForm)
      if (this.addForm.goods_cat.length !== 3) {
        this.addForm.goods_cat = []
      }
    },
    // 监听tab标签的切换
    beforeTabLeave (activeName, oldActiveName) {
    //   console.log('新：' + activeName)
    //   console.log('旧：' + oldActiveName)
      if (oldActiveName === '0' && this.addForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类')
        return false
      }
    },
    async tabClicked () {
      console.log(this.activeIndex)
      // 证明访问的是商品参数面板
      if (this.activeIndex === '1') {
        console.log('动态参数面板')
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'many'
            }
          })
        if (res.meta.status !== 200) {
          this.$message.error('失败')
        }
        console.log(res.data)
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        console.log('静态属性列表')
        const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'only'
            }
          })
        if (res.meta.status !== 200) {
          this.$message.error('失败')
        }
        console.log(res.data)
        this.onlyTableData = res.data
      }
    },
    // 处理图片预览效果
    handlePreview (file) {
      console.log(file)
      this.previewPath = file.response.data.url
      this.previewVisible = true
    },
    // 处理移除
    handleRemove (file) {
    //   console.log(file)
      // 获取将要删除的图片临时地址
      const filePath = file.response.tmp_path
      // 从pics数组中找到这个图片对应的索引值
      const i = this.addForm.pics.findIndex(x => x.pic === filePath)
      //  调用splice方法，将索引值对应的图片信息从数组中删除
      this.addForm.pics.splice(i, 1)
      console.log(this.addForm)
    },
    // 监听图片上传成功的事件
    handleSuccess (response) {
      console.log(response)
      // 拼接得到一个图片信息对象
      const picInfo = {
        pic: response.data.tmp_path
      }
      // 将图片信息对象保存至数组中
      this.addForm.pics.push(picInfo)
      console.log(this.addForm)
    },
    add () {
      console.log(this.addForm)
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) {
          this.$message.error('添加必要的表单项')
        }
        // 执行添加 需要将商品类型从数组转化为逗号连接
        this.addForm.goods_cat = this.addForm.goods_cat.join(',')
        // 处理动态参数
        // console.log(this.manyTableData)
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.addForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals
          }
          this.addForm.attrs.push(newInfo)
        })
        // console.log(this.addForm)
        // 发起请求添加商品 商品的名称必须唯一
        const { data: res } = await this.$http.post('goods', this.addForm)
        console.log(res)
        console.log(res.data)
        if (res.meta.status !== 201) {
          return this.$message.error('失败')
        }
        this.$message.success('添加成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    // 商品分类id
    cateId () {
      if (this.addForm.goods_cat.length === 3) {
        return this.addForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>
<style>
div.el-popper.el-cascader__dropdown {
     top: 450px !important;
}
.el-checkbox {
    margin: 0 5px 0 0 !important;
}
.previewImg {
    width: 100%;
}
.el-button.btnAddForm.el-button--primary {
    margin-top: 15px;
}
</style>
