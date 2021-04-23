<template>
  <div class="login_container">
    <div class="login_box">
      <div class="photo">
        <img src="../assets/logo.png">
      </div>
<!--      rules指向的是校验规则-->
      <el-form ref="loginFormRef" :model="Form_c" :rules="loginFromRules" class="login_form" label-width="0px">
        <el-form-item prop="username">
          <el-input v-model="Form_c.username" prefix-icon="iconfont icon-yonghu"></el-input>
        </el-form-item>
        <el-form-item prop="password">
          <el-input v-model="Form_c.password" prefix-icon="iconfont icon-password" type="password"></el-input>
        </el-form-item>
        <el-form-item class="button_login">
          <el-button type="primary" @click="validateForm">登录</el-button>
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
      Form_c: {
        password: '123456',
        username: 'admin'
      },
      loginFromRules: {
        username: [{
          required: true,
          message: '请输入账号',
          trigger: 'blur'
        }, {
          min: 5,
          max: 11,
          message: '长度在 5 到 11 个字符',
          trigger: 'blur'
        }],
        password: [{
          required: true,
          message: '请输入密码',
          trigger: 'blur'
        }, {
          min: 5,
          max: 11,
          message: '长度在 5 到 11 个字符',
          trigger: 'blur'
        }]
      }
    }
  },
  methods: {
    resetLoginForm () {
      console.log(this)
      this.$refs.loginFormRef.resetFields()
    },
    validateForm () {
      this.$refs.loginFormRef.validate(async valid => {
        console.log(this)
        if (!valid) return
        const { data: result } = await this.$http.post('login', this.Form_c)
        // console.log(result)
        if (result.meta.status !== 200) return this.$message.error('fail login!')
        this.$message.success('successful login ,welcome')
        // console.log(result)
        window.sessionStorage.setItem('token', result.data.token)
        this.$router.push('/home')
      })
    }
  }
}
</script>

<style lang="less" scoped>
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

  .photo {
    position: absolute;
    left: 50%;
    right: 50%;
    transform: translate(-50%, -50%);
    width: 130px;
    height: 130px;
    position: absolute;
    padding: 10px;
    border: 1px solid #eee;
    border-radius: 50%;
    box-shadow: 0 0 10px;
    background-color: #fff;

    img {
      height: 100%;
      width: auto;
      border-radius: 50%;
      background-color: #eee;
    }
  }

  .login_form {
    position: absolute;
    bottom: 0px;
    width: 100%;
    padding: 0 20px;
    box-sizing: border-box;

    .button_login {
      text-align: right;
    }
  }
}

</style>
