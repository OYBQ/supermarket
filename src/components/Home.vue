<template>

  <el-container class="home-container">
    <!-- 头部区域 -->
    <el-header>
      <div>
        <img src="../assets/heima.png" alt="">
        <span>电商管理平台</span>
      </div>
      <el-button type="info" @click="loginOut">退出</el-button>  
    </el-header>


    <!-- 页面主体区域 -->
    <el-container>
      <!-- 侧边栏区域 -->
      <el-aside :width="isCollapse ? '64px' : '200px'" >
          <!-- 顶部的展开折叠按钮 -->
            <div class="toggle-button" @click="toggleCollapse">|||</div>
              <el-menu background-color="#333744 " text-color="#fff" active-text-color="#359BFF" 
              unique-opened :collapse="isCollapse" :collapse-transition="false" router :default-active="activePath2">
            <!-- 一级菜单 -->
            <el-submenu :index="item.id + ''" v-for="item in menulist" :key="item.id">
              <!-- 一级菜单的模板区 -->
              <template slot="title">
                <!-- 图标 -->
                <i :class="iconObj[item.id]"></i>
                <!-- 文本 -->
                <span>{{ item.authName}}</span>
              </template>

              <!-- 二级菜单 -->
              <el-menu-item :index="'/'+ subItem.path + ''"  v-for="subItem in item.children" :key="subItem.id" @click="saveNavState('/'+ subItem.path)">
                <!-- 二级菜单模板区域 -->
                <template slot="title">
                  <!-- 图标 -->
                  <i class="el-icon-menu"></i>
                  <!-- 文本 -->
                  <span>{{ subItem.authName}}</span>
                </template>
            </el-menu-item> 

            </el-submenu>        
          </el-menu>
      </el-aside>


      
      <!-- 右侧内容区域 -->
      <el-main>
        <router-view></router-view>
      </el-main>


    </el-container>
  

    
  </el-container>

  

    

</template>
<script>
export default {
  data() {
    return {
      //左侧菜单数据
      menulist: [],
      iconObj: {
        '125': 'iconfont icon-users',
        '103': 'iconfont icon-tijikongjian',
        '101': 'iconfont icon-shangpin',
        '102': 'iconfont icon-danju',
        '145': 'iconfont icon-baobiao'
      },
      isCollapse: false,
      // 保存被激活链接的地址
      activePath2: '',

    }
  },
  //在页面挂载前获取到数据
  created() {
    this.getMenuList(),
    this.activePath2 = window.sessionStorage.getItem('activePath')
  },
  methods: {
    // 退出登录 销毁sessionStorage中存储的数据
    loginOut() {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
      //获取所有的菜单
    async getMenuList() {
      const { data: res } = await this.$http.get('menus')
      //判断是否获取数据成功
      //如果失败
      if(res.meta.status !== 200) return this.$message.error(res.meta.msg)

      //如果成功,则把值传到data对象中
      this.menulist = res.data
      // console.log(res);
    },

    //展开或者折叠菜单的方法
    toggleCollapse() {
      this.isCollapse = ! this.isCollapse;

    },
    // 保存当前链接的激活状态
    saveNavState(activePath) {
      window.sessionStorage.setItem('activePath', activePath)

    }
    

  },

}
</script>
<style lang="less" scoped>
.home-container {
  height: 100%;
}
.el-header {
  background-color: #373d41;
  padding-left: 0;
  display: flex;
  justify-content: space-between;
  font-size: 20px;
  align-items: center;
  div {
    display: flex;
    align-items: center;
    span {
      margin-left: 15px;
    }
  }
}
.el-aside {
  background-color: #373d44;
  .el-menu {
    border-right: none;
  }
  .toggle-button {
    background-color: #4a5064;
    text-align: center;
    letter-spacing: 0.2em;
    color: #fff;
    cursor: pointer;

  }



}

.el-main {
  background-color: #EAEDF1;
}

.iconfont {
  margin-right: 10px;
}



</style>