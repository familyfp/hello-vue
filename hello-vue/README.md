# 安装环境：vue、node 、mongodb 、express、Vue-Resource

## 1.搭建Vue前端项目
```
//利用vue-cli搭建环境
$ npm install vue
//全局安装 vue-cli
$ npm install --global vue-cli
//创建一个基于 webpack 模板的新项目
$ vue init webpack my-project
//安装依赖，走你
$ cd my-project
$ npm run dev
```

## 2.修改前端文件
App.vue为如下代码
```
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
    data() {
        return {
            account : '',
            password : ''
        }
    },
    methods:{
      login() {
        // 获取已有账号密码
        this.$http.get('/api/login/getAccount')
          .then((response) => {
            // 响应成功回调
            console.log(response)
            let params = {
              account : this.account,
              password : this.password
            };
            // 创建一个账号密码
            return this.$http.post('/api/login/createAccount',params);
          })
          .then((response) => {
            console.log(response)
          })
          .catch((reject) => {
            console.log(reject)
          });
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
```
这时回到浏览器，如无意外应该会出现两个输入框和一个登录按钮，当然现在去点击登录按钮请求接口，是不行的,需要去搭建Node

## 3.安装Node环境
###  自行百度安装node
在项目的根目录新建一个叫server的目录，用于放置Node的东西。进入server目录，再新建三个js文件：
- index.js （入口文件）
- db.js （设置数据库相关）
- api.js （编写接口）
-
## 4.配置后端环境
#### 安装 express 、vue-resource和mongodb(都是在项目文件夹中安装)

```
//安装express
$ npm install express --save
/*安装vue-resource,(这个需要在项目的main.js中引入vue-resource)需要在main.js中添加如下语句
import VueResource form 'vue-resource'
Vue.use(VueResource)*/
$ npm install vue-resource --save
$ npm install mongosse --save
```

index.js中写入代码

```
// 引入编写好的api
const api = require('./api');
// 引入文件模块
const fs = require('fs');
// 引入处理路径的模块
const path = require('path');
// 引入处理post数据的模块
const bodyParser = require('body-parser')
// 引入Express
const express = require('express');
const app = express();

app.use(bodyParser.json());
app.use(bodyParser.urlencoded({extended: false}));
app.use(api);
// 访问静态资源文件 这里是访问所有dist目录下的静态资源文件
app.use(express.static(path.resolve(__dirname, '../dist')))
// 因为是单页应用 所有请求都走/dist/index.html
app.get('*', function(req, res) {
    const html = fs.readFileSync(path.resolve(__dirname, '../dist/index.html'), 'utf-8')
    res.send(html)
})
// 监听8088端口
app.listen(8088);
console.log('success listen…………');
```
db.js中写入代码

```
// Schema、Model、Entity或者Documents的关系请牢记，Schema生成Model，Model创造Entity，Model和Entity都可对数据库操作造成影响，但Model比Entity更具操作性。
const mongoose = require('mongoose');
// 连接数据库 如果不自己创建 默认test数据库会自动生成
mongoose.connect('mongodb://localhost/test');

// 为这次连接绑定事件
const db = mongoose.connection;
db.once('error',() => console.log('Mongo connection error'));
db.once('open',() => console.log('Mongo connection successed'));
/************** 定义模式loginSchema **************/
const loginSchema = mongoose.Schema({
    account : String,
    password : String
});

/************** 定义模型Model **************/
const Models = {
    Login : mongoose.model('Login',loginSchema)
}

module.exports = Models;
```
api.js中写入

```
"use strict";
const models = require('./db');
const express = require('express');
const router = express.Router();

/************** 创建(create) 读取(get) 更新(update) 删除(delete) **************/

// 创建账号接口
router.post('/api/login/createAccount',(req,res) => {
    // 这里的req.body能够使用就在index.js中引入了const bodyParser = require('body-parser')
    let newAccount = new models.Login({
        account : req.body.account,
        password : req.body.password
    });
    // 保存数据newAccount数据进mongoDB
    newAccount.save((err,data) => {
        if (err) {
            res.send(err);
        } else {
            res.send('createAccount successed');
        }
    });
});
// 获取已有账号接口
router.get('/api/login/getAccount',(req,res) => {
    // 通过模型去查找数据库
    models.Login.find((err,data) => {
        if (err) {
            res.send(err);
        } else {
            res.send(data);
        }
    });
});

module.exports = router;
```
进入server目录，敲上 node index命令，node就会跑起来，这时在浏览器输入http://localhost:8088/api/login/getAccount就能访问到这个接口了

回到localhost:8080前端项目请求接口，发现还是不能访问数据，这是因为使用的端口号不一致，导致了跨域，需要解决跨域问题；

在项目中有一个config文件夹，里面有一个index.js文件，修改

```
 proxyTable: {
        '/api': {
        target: 'http://localhost:8088/api/',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
```
**注意：修改了proxyTable后需要重新npm run dev ;如果你电脑上启动了nginx代理，请先关闭，不然无法访问通后端代码**

访问前端代码，发送请求可查看请求后端成功；
