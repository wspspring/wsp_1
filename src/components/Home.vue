<template>
<el-container class="home-container">
    <!-- 头部 -->
    <el-header>
        <div>
            <img src="../assets/logo.png" alt="" class="logoImg">
            <span>媒体融合与传播国家重点实验室 科研平台</span>
        </div>
        <!-- 退出按钮 -->
        <el-button type="info" @click="logout">退出</el-button>
    </el-header>
    <!-- 页面主体区域 -->
    <el-container>
        <!-- 左边栏 -->
        <el-aside :width="isCollapse ? '64px' : '200px'">
            <div class="toggle-button" @click="toggleCollapse">
                |||
            </div>
            <el-menu
            background-color="#545c64"
            text-color="#fff"
            active-text-color="#409eff"
            unique-opened
            :collapse='isCollapse'
            :collapse-transition="false"
            router
            :default-active="$route.path">
            <!-- 一级菜单 -->
                <el-submenu :index="item.id + ''" v-for="item in menuList" :key="item.id">
                    <!-- 一级菜单的模板区域 -->
                    <template slot="title">
                        <!-- 图标 -->
                        <i :class="iconObj[item.id]"></i>
                        <!-- 文本 -->
                        <span>{{item.authName}}</span>
                    </template>
                    <!-- 二级菜单 -->
                    <el-menu-item :index="'/'+ subItem.path" v-for="subItem in item.children" :key="subItem.id">
                        <template slot="title">
                        <!-- 图标 -->
                        <i class="el-icon-menu"></i>
                        <!-- 文本 -->
                        <span>{{subItem.authName}}</span>
                    </template>
                    </el-menu-item>
                </el-submenu>
            </el-menu>
        </el-aside>
        <!-- 右边栏 -->
        <el-main>
            <!-- 路由占位符 -->
            <router-view></router-view>
        </el-main>
    </el-container>
</el-container>
</template>

<script>
export default {
  data () {
    return {
      menuList: [],
      iconObj: {
        125: 'el-icon-user',
        103: 'el-icon-s-check',
        101: 'el-icon-s-goods',
        102: 'el-icon-s-order',
        145: 'el-icon-s-marketing'
      },
      isCollapse: false
    }
  },
  created () {
    this.getMenuList()
  },
  methods: {
    //   退出登录
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 获取菜单 解构赋值
    async getMenuList () {
      const { data: res } = await this.$http.get('menus')
      console.log(res)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      this.menuList = res.data
    },
    // 菜单收起
    toggleCollapse () {
      this.isCollapse = !this.isCollapse
    }
  }
}
</script>
<style lang='less' scoped>
.el-header {
    background-color: #373d41;
    display: flex;
    justify-content: space-between;
    padding-left: 0;
    align-items: center;
    color: #fff;
    font-size: 20px;
    > div {
        display: flex;
        // 垂直居中
        align-items: center;
    }
    > span {
        margin-left: 20px;
    }
}
.el-aside {
    background-color: #333744;
    .el-menu {
        border-right: none;
    }
}
.el-main {
    background-color: #eaedf1;
}
.home-container {
    height: 100%;
}
.logoImg {
    width: 50px;
    height: 50px
}
.toggle-button {
    background-color: #4a5064;
    font-size: 15px;
    text-align: center;
    color: #fff;
    letter-spacing: 0.2em;
    cursor: pointer;
}
</style>
