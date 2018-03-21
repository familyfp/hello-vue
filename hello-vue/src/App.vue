<template>
  <div id="app">
    <input class="form-control" id="inputEmail3" placeholder="请输入账号" v-model="account">
    <input type="password" class="form-control" id="inputPassword3" placeholder="请输入密码" v-model="password">
    <button type="submit" class="btn btn-default" @click="login">登录</button>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      account: '',
      password: ''
    }
  },
  methods: {
    login () {
      // 获取已有账号密码
      this.$http.get('/api/login/getAccount')
        .then((response) => {
          // 响应成功回调
          console.log(response)
        })
        .catch((reject) => {
          console.log(reject)
        })
    }
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
