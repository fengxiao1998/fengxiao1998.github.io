---
title: 前端node跨域解决方案cors以及jsonp
date: 2022-06-17 15:03:18
tags: [node,前端跨域,cors,JSONP]
---

## 前言

前端跨域问题是个老生常谈的问题，面试时也会千篇一律的回答，比较流行的axios+代理的方式，还有原始的cors和jsonp，今天我们用node实现cors和jsonp的跨域方式。

## 跨域与请求

#### 跨域

基本的概念不过多陈申，造成的原因无非是**非同源策略**，即**协议、域名、端口号**不同。

#### 请求

网络请求这种简单的内容不过多陈述，只进行一个小的有趣知识点进行说明，**简单请求**和**复杂请求**。

##### 简单请求特征：

1. 请求方式为**GET** 、**POST**、**HEAD**三者之一
2. HTTP头部信息不超过以下几种字段：
   1. **Accept**：告诉WEB服务器自己接受什么介质类型，**/**表⽰任何类型，**type/*** 表⽰该类型下的所有⼦类型，type/sub-type。
   2. **Accept-Language**：浏览器申明⾃⼰接收的语⾔语⾔跟字符集的区别：中⽂是语⾔，中⽂有多种字符集，⽐如big5，gb2312，gbk等等。
   3. **Content-Language**：WEB 服务器告诉浏览器⾃⼰响应的对象的语⾔。
   4. **Content-Type**：用于指示资源的原始媒体类型，其中它的值只限于以下三者之一
      - text/plain
      - multipart/form-data
      - application/x-www-http-urlencoded
3. 客户端与服务器**只会发起一次请求**

##### 复杂请求特征：

1. 请求方式为**GET** 、**POST**、**HEAD**三者之外
2. 请求头中包含自定义头部字段
3. 向服务器发送了application/json格式的数据
4. 浏览器与服务器正式通信之前，浏览器会先发送 **OPTION** 请求进行预检，以获知服务器是否允许该实际请求，服务器成功响应后，才会发送真正的请求，因此它**会发送两次请求**。

#### 请求实验

通过在测试html文件中发起**DELETE**请求，并在node端进行接受，在点击按钮指向ajax操作后，可以在浏览器网络请求中看到以下信息，说明网络请求确实执行了两次，为复杂请求。

![image-20220617161535826](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220617161535826.png)

## cors解决跨域

#### 概念

先放一张笔者视频学习时截取的图片对CORS进行系统说明：

![550923fd77801650bdabe6a4e594aef](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/550923fd77801650bdabe6a4e594aef.png)

接下来通过express+cors演示解决跨域问题，新建文件夹，输入 `npm i  express cors`安装express和cors包，在同级根目录下新建index.js和index.html两个文件，index.js用于创建服务器以及编写接口，index.html用于测试跨域问题。
index.js下写入：

```javascript
const express = require('express')
const app = express()

// CORS 解决跨域方案，一定要放在路由之前
const cors = require('cors')
app.use(cors())

//如果是post请求，需要挂载解析请求体的中间件
app.use(express.urlencoded({ extended: false }))//解析urlencoded
app.use(express.json())//解析json

app.get('/get', (req, res) => {
  const query = req.query
  res.send({
    status: 0,
    msg: 'Get请求成功',
    data: query
  })
})

app.post('/post', (req, res) => {
  const body = req.body
  res.send({
    status: 0,
    msg: 'Post请求成功',
    data: body
  })
})

app.listen(80)
```

index.html下写入：

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script src="https://cdn.staticfile.org/jquery/3.4.1/jquery.min.js"></script>
</head>

<body>
  <button class="get" id="get">GET</button>
  <button class="post" id="post">POST</button>
  <script>
    $(function () {
      $('#get').on('click', function () {
        $.ajax({
          type: 'GET',
          url: 'http://127.0.0.1/api/get',
          data: {
            name: '这是跨域问题'
          },
          success: function (res) {
            console.log(res)
          }
        })
      })

      $('#post').on('click', function () {
        $.ajax({
          type: 'POST',
          url: 'http://127.0.0.1/api/post',
          data: {
            age: '这是跨域问题'
          },
          success: function (res) {
            console.log(res)
          }
        })
      })
    })
  </script>
</body>
</html>
```

在index.html文件中通过live server插件打开(快捷键alt+L+O,没有的话可以去vscode拓展市场下载一个)，在当前文件夹下打开cmd，输入`node .\index.js`将我们的服务跑起来，此时我们的客户端地址为：localhost:5500，服务器端地址为：localhost:80。端口号不同，满足非同源策略。接下来在页面中点击get 和 post按钮，在浏览器网络请求中查看是否请求成功。

![image-20220617163204035](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220617163204035.png)

可以看到，请求成功，不妨在index.js中将以下内容注释掉，重新启动node服务，查看还能否请求成功。

```javascript
const cors = require('cors')
app.use(cors())
```

代码注释后，可以看到浏览器控制台抛出以下错误，错误即为跨域请求失败，说明以上两行代码的的确解决了跨域问题。

![image-20220617163349772](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220617163349772.png)

## JSONP解决跨域问题

#### 概念与特点

##### 概念：

浏览器通过**script**标签的**src**属性，请求服务器上的数据，同时服务器返回一个函数的调用,这种请求方式叫做JSONP。

##### 特点：

1. JSONP不属于真正的Ajax请求，因为他没有使用XMLHttpRequest这个对象
2. JSONP**只支持GET**请求

#### 实现步骤

1. 获取客户端发送过来的回调函数的名字
2. 得到要通过JSONP形式发送给客户端的数据
3. 根据前两步得到的数据，拼接出一个函数调用的字符串
4. 把上一步拼接得到的字符串，响应给客户端的**script**标签进行解析执行 

#### 具体操作

在上文使用的index.js文件中新写入以下内容：

```javascript
//必须在cors之前配置JSONP，要把这段放在app.use(cors())之前，否则跨域就不是通过JSONP执行得了。

app.get('/api/jsonp', (req, res) => {
  //todo 定义JSONP接口具体实现过程
  //1.获取客户端发送过来的回调函数的名字
  const funcName = req.query.callback
  //2.得到要通过JSONP形式发送给客户端的数据
  const data = { name: '张双' }
  //3.拼接出一个函数的调用
  const scriptStr = `${funcName}(${JSON.stringify(data)})`
  //4.把拼接的字符串相应给客户端
  res.send(scriptStr)

})
```

在index.html中新增以下内容：

```html
  <button class="delete" id="deleteBtn">delete</button>
  <script>
	//测试JSONP
      $('#jsonpBtn').on('click', function () {
        $.ajax({
          type: 'GET',
          url: 'http://127.0.0.1/api/jsonp',
          dataType: 'jsonp',//表示要发起 JSONP请求
          data: {
            age: '这是JSONP请求'
          },
          success: function (res) {
            console.log(res)
          }
        })
      })
  </script>

```

点击jsonp按钮，在控制台看到以下信息，说明数据被返回了。
![image-20220617165546405](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220617165546405.png)

再看看网络请求一栏，如果你再点击一下GET按钮，同时将网络中的筛选项调至Fetch/XHR,你会发现只有get请求显示了出来，这也从侧面验证了**JSONP不属于真正的Ajax请求，因为他没有使用XMLHttpRequest这个对象**这个特点。

![image-20220617165847778](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220617165847778.png)



## 总结说明

目前项目的开发大多数是前后端分类，前端由于vue,react等框架的现世，跨域方式绝大多数也成为了**Axios+代理**的方案，传统的方案cors和jsonp逐渐没落，jsonp因为自身的缺陷(只支持get请求等)基本已经无人使用。技术更替是日新月异，对于未来的跨域解决方案笔者很是期待。