<!-- 组件说明 -->
<template>
  <div class="login_container">
    <div class="login_box">
      <!-- 用户头像区域 -->
      <div class="avatar_box">
        <img src="../assets/logo.png" alt />
      </div>
      <!-- 表单区域 -->
      <el-form
        label-width="0px"
        class="login_form"
        :model="loginFrom"
        :rules="loginFromRules"
        ref="loginFormRef"
      >
        <!-- 用户名 -->
        <el-form-item prop="username">
          <el-input prefix-icon="iconfont icon-users" v-model="loginFrom.username"></el-input>
        </el-form-item>
        <!-- 密码 -->
        <el-form-item prop="password">
          <el-input
            prefix-icon="iconfont icon-3702mima"
            v-model="loginFrom.password"
            type="password"
          ></el-input>
        </el-form-item>
        <!-- 按钮 -->
        <el-form-item class="btns">
          <el-button type="primary" @click="login">登录</el-button>
          <el-button type="info" @click="resetLoginForm">重置</el-button>
        </el-form-item>
      </el-form>
    </div>
  </div>
</template>

<script>

export default {
  components: {},
  data() {
    return {
      // 这是表单中的数据绑定对象
      loginFrom: {
        username: 'admin',
        password: '123456'
      },
      // 这是表单中的数据验证规则对象
      loginFromRules: {
        username: [
          { required: true, message: '请输入登录用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入登录密码', trigger: 'blur' },
          { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  computed: {},
  methods: {
    // 点击重置按钮
    resetLoginForm() {
      // console.log(this.);
      this.$refs.loginFormRef.resetFields()
    },
    //表单提交前的预验证,验证通过后才能发送到哦服务器中
    login() {
      this.$refs.loginFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('login', this.loginFrom)
        // console.log(res);
        if (res.meta.status !== 200) return this.$message.error('登录失败')
        this.$message.success('登录成功')

        //1 将登录成功之后的token，保存到客户端的sessionStorage中
        // console.log(res)

        //1.1 项目中除了登录之外的其他API接口，必须 要在登录之后才能访问
        // 1.2 token只应当在当前网站打开期间生效，所以必须要将token保存在sessionStorage
        window.sessionStorage.setItem('token', res.data.token)
        //2 通过编程式导航跳转到后台主页，路由地址为/home
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang='less' scoped>
//@import url() 加上scope就只在当前组件内生效
.login_container {
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

  .avatar_box {
    height: 130px;
    width: 130px;
    background-color: pink;
    border-radius: 50%;
    box-shadow: 0 0 10px #ddd;
    position: absolute;
    left: 50%;
    transform: translate(-50%, -50%);

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

    .btns {
      display: flex;
      justify-content: flex-end;
    }
  }
}


</style>