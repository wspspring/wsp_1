<template>
    <div class="login_container">
        <div class="login_box">
            <!-- 头像区域 -->
            <div class="avatar_box">
                <img src="../assets/logo.png" alt="">
            </div>
            <!-- 表单区域 -->
            <el-form ref="loginFormRef" class="login_form" :model="loginForm" :rules="loginFormRules">
                <!-- 用户名 -->
                <el-form-item prop="username">
                    <el-input v-model="loginForm.username" prefix-icon="el-icon-user-solid"></el-input>
                </el-form-item>
                <!-- 密码 -->
                <el-form-item prop="password">
                    <el-input v-model="loginForm.password" prefix-icon="el-icon-s-goods" type="password"></el-input>
                </el-form-item>
                <!-- 按钮区域 -->
                <el-form-item class="btns">
                    <el-button type="primary" @click="login">确认</el-button>
                    <el-button type="info" @click="resetLoginForm">重置</el-button>
                </el-form-item>
            </el-form>
        </div>
    </div>
</template>
<script>
export default {
  data () {
    return {
    // 登录表单的数据绑定对象
      loginForm: {
        username: '',
        password: ''
      },
      // 表单验证规则
      loginFormRules: {
        //   验证用户名
        username: [
          { required: true, trigger: 'blur', message: '请输入用户名' },
          { min: 3, max: 20, message: '长度在3和20之间', trigger: 'blur' }
        ],
        //   验证密码
        password: [
          { required: true, trigger: 'blur', message: '请输入密码' },
          { min: 3, max: 15, message: '长度在3和15之间', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    //   表单重置函数
    resetLoginForm () {
    //   console.log(this.$refs)
      this.$refs.loginFormRef.resetFields()
    },
    // 表单确认提交
    login () {
      this.$refs.loginFormRef.validate(async valid => {
        //   表单验证结果 valid
        // console.log(valid)
        if (!valid) {
          return console.log('验证失败')
        }
        const { data: res } = await this.$http.post('login', this.loginForm)
        console.log(res)
        if (res.meta.status !== 200) {
          return this.$message.error('登录失败')
        }
        this.$message.success('登录成功')
        console.log(res)
        // 保存token值
        window.sessionStorage.setItem('token', res.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>
<style lang="less" scoped>
.login_container{
    background-color: #2b4b6b;
    height: 100%;
}
.login_box {
    width: 450px;
    height: 300px;
    background-color: #fff;
    border-radius: 3px;
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);

}
.avatar_box {
    width: 100px;
    height: 100px;
    border: 1px solid #eee;
    border-radius: 50%;
    padding: 10px;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: #fff;
    img {
        width: 100%;
        height: 100%;
        border-radius: 50%;
        background-color: #eee;
    }
}
.login_form {
    position: absolute;
    bottom: 0;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;
}
.btns {
    display: flex;
    justify-content: flex-end;
}
</style>
