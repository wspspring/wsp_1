<template>
    <div>
        <!-- 面包屑导航区 -->
        <el-breadcrumb separator-class="el-icon-arrow-right">
        <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
        <el-breadcrumb-item>用户管理</el-breadcrumb-item>
        <el-breadcrumb-item>用户列表</el-breadcrumb-item>
        </el-breadcrumb>
        <el-card class="box-card">
            <!-- 搜索与添加区域 -->
            <el-row :gutter="20">
                <el-col :span="8">
                    <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList">
                        <el-button slot="append" icon="el-icon-search" @click="getUserList"></el-button>
                    </el-input>
                </el-col>
                <el-col :span="4">
                    <el-button type="primary" @click="addDialogVisiable = true">添加用户</el-button>
                </el-col>
            </el-row>
            <!-- 用户列表区域 -->
            <el-table :data="userList" stripe border>
                <el-table-column type="index" label="#"></el-table-column>
                <el-table-column label="姓名" prop="username"></el-table-column>
                <el-table-column label="邮箱" prop="email"></el-table-column>
                <el-table-column label="电话" prop="mobile"></el-table-column>
                <el-table-column label="角色" prop="role_name"></el-table-column>
                <el-table-column label="状态" prop="mg_state">
                    <template slot-scope="scope">
                        <el-switch v-model="scope.row.mg_state" @change="userStateChange(scope.row)"></el-switch>
                    </template>
                </el-table-column>
                <el-table-column label="操作" width="180px">
                    <template slot-scope="scope">
                        <!-- 修改按钮 -->
                        <el-button type="primary" icon="el-icon-edit" size="mini" @click="showEditDialog(scope.row.id)"></el-button>
                        <!-- 删除按钮 -->
                        <el-button type="danger" icon="el-icon-delete" size="mini" @click="removeUserById(scope.row.id)"></el-button>
                        <!-- 分配角色按钮 -->
                        <el-tooltip effect="dark" content="分配角色提示" placement="top" :enterable="false">
                            <el-button type="warning" icon="el-icon-setting" size="mini" @click="setRole(scope.row)"></el-button>
                        </el-tooltip>
                    </template>
                </el-table-column>
            </el-table>
            <!-- 分页区域 -->
            <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :current-page="queryInfo.pagenum"
            :page-sizes="[2, 5, 8]"
            :page-size="queryInfo.pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="total">
            </el-pagination>
            <!-- 添加用户对话框 -->
            <el-dialog
            title="添加用户"
            :visible.sync="addDialogVisiable"
            width="50%"
            @close="addDialogClosed">
                <!-- 内容主体区域 -->
                <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="70px">
                    <el-form-item label="用户名" prop="username">
                        <el-input v-model="addForm.username"></el-input>
                    </el-form-item>
                    <el-form-item label="密码" prop="password">
                        <el-input v-model="addForm.password"></el-input>
                    </el-form-item>
                    <el-form-item label="邮箱" prop="email">
                        <el-input v-model="addForm.email"></el-input>
                    </el-form-item>
                    <el-form-item label="手机" prop="mobile">
                        <el-input v-model="addForm.mobile"></el-input>
                    </el-form-item>
                </el-form>
                <!-- 底部区域 -->
                <span slot="footer" class="dialog-footer">
                    <el-button @click="addDialogVisiable = false">取 消</el-button>
                    <el-button type="primary" @click="addUser">确 定</el-button>
                </span>
            </el-dialog>
            <!-- 编辑对话框 -->
            <el-dialog
            title="修改用户信息"
            :visible.sync="editDialogVisiable"
            width="50%"
            @close="editDialogClosed">
            <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="70px">
                <el-form-item label="用户名称">
                    <el-input v-model="editForm.username" disabled></el-input>
                </el-form-item>
                <el-form-item label="邮箱" prop="email">
                    <el-input v-model="editForm.email"></el-input>
                </el-form-item>
                <el-form-item label="手机号" prop="mobile">
                    <el-input v-model="editForm.mobile"></el-input>
                </el-form-item>
            </el-form>
            <span slot="footer" class="dialog-footer">
                <el-button @click="editDialogVisiable = false">取 消</el-button>
                <el-button type="primary" @click="editUserInfo">确 定</el-button>
            </span>
            </el-dialog>
            <!-- 分配角色对话框 -->
            <el-dialog
            title="分配角色"
            :visible.sync="setRoleDialogVisiable"
            width="50%" @close="setRoleDialogClosed">
            <div>
              <p>当前的用户：{{userInfo.username}}</p>
              <p>当前的角色：{{userInfo.role_name}}</p>
              <p>分配新角色：
                <el-select v-model="selectedRoleId" placeholder="请选择">
                  <el-option
                    v-for="item in rolesList"
                    :key="item.id"
                    :label="item.roleName"
                    :value="item.id">
                  </el-option>
                </el-select>
              </p>
            </div>
            <span slot="footer" class="dialog-footer">
              <el-button @click="setRoleDialogVisiable = false">取 消</el-button>
              <el-button type="primary" @click="saveRoleInfo">确 定</el-button>
            </span>
          </el-dialog>
        </el-card>
    </div>
</template>
<script>
export default {
  data () {
    //   校验邮箱
    var checkEmail = (rule, value, callback) => {
      // 验证邮箱的正则
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if (regEmail.test(value)) {
        //   合法邮箱
        return callback()
      }
      callback(new Error('请输入合法邮箱'))
    }
    // 校验手机号
    var checkMobile = (rule, value, callback) => {
    // 验证手机号的正则
      const regMobile = /^(0|86|17951)?(13[0-9]|15[0123456789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        //   合法手机号
        return callback()
      }
      callback(new Error('请输入合法手机号'))
    }
    return {
    //   获取用户列表的参数对象
      queryInfo: {
        query: '',
        // 当前页大小
        pagesize: 2,
        // 当前页
        pagenum: 1
      },
      userList: [],
      total: 0,
      // 控制添加用户对话框的显示与隐藏，默认隐藏
      addDialogVisiable: false,
      // 添加用户的表单对象
      addForm: {},
      // 添加用户的表单验证规则对象
      addFormRules: {
        username: [
          { required: true, trigger: 'blur', message: '请输入用户名' },
          { min: 3, max: 10, message: '用户名称在3和10之间', trigger: 'blur' }
        ],
        password: [
          { required: true, trigger: 'blur', message: '请输入密码' },
          { min: 6, max: 15, message: '密码在6和15之间', trigger: 'blur' }
        ],
        email: [
          { required: true, trigger: 'blur', message: '请输入邮箱' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, trigger: 'blur', message: '请输入手机号' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制修改对话框的显示与隐藏
      editDialogVisiable: false,
      //   查询到的用户信息
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        email: [
          { required: true, trigger: 'blur', message: '请输入邮箱地址' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, trigger: 'blur', message: '请输入手机号' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      // 控制分配角色对话框的显示与隐藏
      setRoleDialogVisiable: false,
      // 用户信息
      userInfo: {},
      // 角色列表
      rolesList: [],
      // 已经选中的角色id
      selectedRoleId: ''
    }
  },
  created () {
    this.getUserList()
  },
  methods: {
    async getUserList () {
      const { data: res } = await this.$http.get('users', { params: this.queryInfo })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取失败')
      }
      this.userList = res.data.users
      this.total = res.data.total
    },
    // 监听pageSize改变事件
    handleSizeChange (newSize) {
      console.log(newSize)
      this.queryInfo.pagesize = newSize
      this.getUserList()
    },
    // 监听页码
    handleCurrentChange (newPage) {
      console.log(newPage)
      this.queryInfo.pagenum = newPage
      this.getUserList()
    },
    // 监听switch状态的改变
    async userStateChange (userInfo) {
      console.log(userInfo)
      const { data: res } = await this.$http.put(`/users/${userInfo.id}/state/${userInfo.mg_state}`)
      console.log(res)
      if (res.meta.status !== 200) {
        userInfo.mg_state = !userInfo.mg_state
        return this.$message.error('更新用户状态失败')
      }
      this.$message.success('更新用户状态成功')
    },
    // 监听添加对话框的关闭
    addDialogClosed () {
      this.$refs.addFormRef.resetFields()
    },
    addUser () {
      this.$refs.addFormRef.validate(async valid => {
        console.log(valid)
        if (!valid) {
          return this.$message.error('验证失败')
        }
        // 添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.addForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败')
        }
        console.log(res)
        // 隐藏添加用户的对话框
        this.addDialogVisiable = false
        this.getUserList()
      })
    },
    // 添加编辑用户对话框
    async showEditDialog (id) {
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('获取用户信息失败')
      }
      this.editForm = res.data
      this.editDialogVisiable = true
      console.log(this.editForm)
    },
    // 监听用户修改的关闭事件
    editDialogClosed () {
      this.$refs.editFormRef.resetFields()
    },
    // 修改用户信息
    editUserInfo () {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('验证失败')
        }
        const { data: res } = await this.$http.put('users/' + this.editForm.id, { email: this.editForm.email, mobile: this.editForm.mobile })
        console.log(res)
        if (res.meta.status !== 200) {
          this.$meassage.error('修改用户信息失败')
        }
        // 隐藏对话框
        this.editDialogVisiable = false
        // 重新获取数据
        this.getUserList()
        // 提示用户修改成功
        this.$message.success('修改成功')
      })
    },
    // 删除用户信息
    async removeUserById (id) {
      // 询问用户是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      console.log(confirmResult)
      //   如果用户确认删除，返回 confirm，如果用户取消删除，返回 cancel
      if (confirmResult !== 'confirm') {
        return this.$message.info('取消删除！')
      }
      // console.log('确认删除')
      const { data: res } = await this.$http.delete('users/' + id)
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功')
      this.getUserList()
    },
    // 分配角色对话框
    async setRole (userInfo) {
      this.userInfo = userInfo
      // 展示对话框之前，获取所有的角色列表
      const { data: res } = await this.$http.get('roles')
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('获取角色列表失败')
      }
      this.rolesList = res.data
      console.log(this.rolesList)
      this.setRoleDialogVisiable = true
    },
    async saveRoleInfo () {
      if (!this.selectedRoleId) {
        return this.$message.error('请选择角色')
      }
      const { data: res } = await this.$http.put(`users/${this.userInfo.id}/role`, { rid: this.selectedRoleId })
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error('添加用户角色失败')
      }
      this.$message.success('成功')
      this.getUserList()
      this.setRoleDialogVisiable = false
    },
    setRoleDialogClosed () {
      this.userInfo = {}
      this.selectedRoleId = ''
    }
  }
}
</script>
<style scoped>

</style>
