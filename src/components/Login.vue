<template>
<div class="login-page">
<el-row align="center" justify="center" class="wh-header">
  <el-col :span="24"><h3 >nCoV 武汉孕妇支援-管理后台</h3></el-col>
</el-row>
<el-row align="center" justify="center">
  <el-col :span="16" :offset="1">
    <el-input placeholder="请输入手机号" v-model="mobile" class="wh-input-width">
        <template slot="prepend">手机号</template>
    </el-input>
  </el-col>
  <el-col :span="4" :offset="1">
    <el-button v-on:click="getVerifycode" v-model="verifyCodeTime">{{this.verifyCodeTime}}</el-button>
  </el-col>
</el-row>
<el-row align="center" justify="center">
  <el-col :span="16" :offset="1">
    <el-input placeholder="短信验证码" v-model="smsverifycode" :disabled="!smsstate" class="wh-input-width">
        <template slot="prepend">验证码</template>
    </el-input>
  </el-col>
</el-row>
<el-row align="center" justify="center">
  <el-col :span="24">
    <el-button :disabled="!smsstate" v-on:click="loginWithCode" class="wh-btn-login">登录</el-button>
  </el-col>
</el-row>
<el-dialog
  title="登陆提示"
  :visible.sync="msgModel"
  width="50%"
  center>
  <span>{{errorMsg}}</span>
  <span slot="footer" class="dialog-footer">
    <el-button @click="msgModel = false">取 消</el-button>
    <el-button type="primary" @click="msgModel = false">确 定</el-button>
  </span>
</el-dialog>
</div>
</template>

<script>
export default {
  name: 'Login',
  data () {
    return {
      mobile: '',
      verifyCodeTime: '获取',
      timer: 60,
      interval: '',
      smsverifycode: '',
      smsstate: false,
      msgModel: false,
      errorMsg: ''
    }
  },
  methods: {
    getVerifycode () {
      var params = {'phone': this.mobile}
      let that = this
      this.$http.post('/wh/admin/sms', params)
        .then(function (response) {
          console.log(response.data)
          if (response.data.code === '0000') {
            that.smsstate = true
            that.interval = setInterval(that.getVerifyCodeTime, 1000)
          }
        })
        .catch(function (error) {
          console.log(error)
        })
    },
    getVerifyCodeTime () {
      this.timer--
      if (this.timer < 10) {
        this.verifyCodeTime = '0' + this.timer + 'S'
      } else {
        this.verifyCodeTime = this.timer + 'S'
      }
      if (this.timer === 0 || this.timer < 0) {
        clearInterval(this.interval)
        this.smsstate = false
        this.verifyCodeTime = '获取'
        this.timer = 60
      }
    },
    loginWithCode () {
      let that = this
      var params = {'phone': this.mobile,
        'verifyCode': this.smsverifycode}
      this.$http.post('/wh/admin/login', params)
        .then(function (response) {
          console.log(response.data)
          if (response.data.code === '0000') {
            localStorage.setItem('token', response.data.result.token)
            localStorage.setItem('tokendate', new Date())
            that.$router.push({ 'name': 'EndIndex' })
          } else {
            that.msgModel = true
            that.errorMsg = response.data.msg
          }
        })
        .catch(function (error) {
          console.log(error)
        })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
  .login-page {
    position: absolute;
    top: 20%;
    left: 0;
    right: 0;
    margin: 20px;
    border: 1px #ddd solid;
  }
  .wh-header {
    background: #4093ff;
    color: white;
  }
  .wh-input-width {
    min-width: 200px;
    max-width: 300px;
  }
  .wh-btn-login {
    width: 200px;
  }
  .el-row {
    margin-bottom: 20px;
    &:last-child {
      margin-bottom: 0;
    }
  }
  .el-col {
    border-radius: 4px;
  }
  .bg-purple-dark {
    background: #99a9bf;
  }
  .bg-purple {
    background: #d3dce6;
  }
  .bg-purple-light {
    background: #e5e9f2;
  }
  .grid-content {
    border-radius: 4px;
    min-height: 36px;
  }
  .row-bg {
    padding: 10px 0;
    background-color: #f9fafc;
  }
</style>
