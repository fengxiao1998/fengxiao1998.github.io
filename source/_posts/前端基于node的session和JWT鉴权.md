---
title: 前端基于node的Session和JWT鉴权
date: 2022-07-05 15:03:18
tags: [node,鉴权,token,Session,JWT]
---

## 前言

关于前端鉴权登录是比较常见的需求了，本文将从服务端渲染和前后端分离的不同角度下演示鉴权。

源码：https://github.com/fengxiao1998/SessionAndJWT.git

(心态崩了，写了一半了编辑器崩了，我还没保存  :(

## 服务端渲染及session鉴权

#### 服务端渲染

服务端渲染简单来说就是前端页面是由服务器通过字符串拼接动态生成的，客户端不需要额外通过Ajax请求参数，只需要做好渲染工作即可。

##### 优点

1. 前端耗时少，前端只需要请求一次接口就能将数据渲染出来，首屏加载速度变快。
2. 利于SEO，因为服务器端相应的是完整的html页面内容，利于爬虫获取信息。

##### 缺点

1. 占用服务器资源，请求过多会造成访问压力。
2. 不利于前后端分类，并且前端复杂度高时不利于开发。

#### 服务端身份验证Session原理

对于服务端渲染，推荐使用Session认证机制，再次之前，先说明一下cookie

![df50f2d722ss6f259030db911ee21b1ad](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/df50f2d722ss6f259030db911ee21b1ad.png)

比如你可以在baidu.com看到以下cookie：

![image-20220627092511922](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220627092511922.png)



session的鉴权就是利用了cookie，用户调用登录接口，完成账号密码的校验之后，将用户信息或者其他校验信息生成为cookie字符串，返回给用户，同时将cookie存储在服务器内存，用户请求其他接口时，会在请求头自动将cookie发送给服务器，服务器会通过与服务器内存中的用户信息匹配，如果匹配成功，则返回客户端想要的内容，否则抛出错误提示客户端需要重新登录。大致流程图如下：

![image-20220704153508473](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220704153508473.png)

#### 实践操作

接下来我们来进行实践操作，在此之前请预先执行 `npm i express express-session`，安装所需要的模块。

index.js文件的代码如下：

```javascript
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()

// 01：配置 Session 中间件
const session = require('express-session')
app.use(
  session({
    secret: 'heyyyyfx',//此处的secret密钥可以是任意字符串，是你自己制定的专属加密方案，此处笔者将以自己的名字为例
    resave: false,//无需在意，但是要写上
    saveUninitialized: true,//无需在意，但是要写上
  })
)

// 托管静态页面，此处笔者代理了一个静态文件，文件内容下文可见。
app.use(express.static('./pages'))
// 解析 POST 提交过来的表单数据
app.use(express.urlencoded({ extended: false }))

// 登录的 API 接口
app.post('/api/login', (req, res) => {
  // 判断用户提交的登录信息是否正确，此处写死一个账号密码校验，在实际开发中肯定是需要数据库匹配。
  if (req.body.username !== 'admin' || req.body.password !== '000000') {
    return res.send({ status: 1, msg: '登录失败' })
  }

  // 02：请将登录成功后的用户信息，保存到 Session 中
  // 注意：只有成功配置了 express-session 这个中间件之后，才能够通过 req 点出来 session 这个属性
  req.session.user = req.body // 用户的信息，我们将用户的信息转换成cookie字符串返回给用户。
  req.session.islogin = true // 用户的登录状态，也是我们鉴权的参考

  res.send({ status: 0, msg: '登录成功' })
})

// 获取用户姓名的接口
app.get('/api/username', (req, res) => {
  // 03：请从 Session 中获取用户的名称，响应给客户端
  if (!req.session.islogin) {//此处就进行了鉴权，看用户的cookie是否有我们之前发送给他的islogin字段。
    return res.send({ status: 1, msg: 'fail' })
  }
  res.send({
    status: 0,
    msg: 'success',
    username: req.session.user.username,
  })
})

// 退出登录的接口
app.post('/api/logout', (req, res) => {
  // 04：清空 Session 信息
  req.session.destroy()
  res.send({
    status: 0,
    msg: '退出登录成功',
  })
})

// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(80, function () {
  console.log('Express server running at http://127.0.0.1:80')
})

```

笔者在此只附上index.js的内容，其他文件内容可以在**前言**中拿到源码。

#### 其他

##### 缺陷

可以看到，Session机制需要cookie的配合才能实现，因此cookie的的缺点或特性也就会影响到Session鉴权，比如，cookie是默认不支持跨域的，当前端跨域请求后端接口时，**需要做很多额外的配置**，这也就是为什么Session推荐在服务端使用。

##### 关于跨域

笔者在本文中说到的的跨域问题，指的是客户端和服务端二者的跨域，如果读者下载了源码，可以看到笔者是在app.js(index.js)中使用`app.use(express.static('./pages'))`进行了静态托管，以此来保证客户端和服务端都是locallhost:80，是同源的。感兴趣的读者可以尝试用**live Sever**来代理Index.html文件，看看效果如何，在此之前记得引入`cors`中间件支持跨域。

#### 想说的

其实笔者在此只是简单讲解了Session鉴权的大致原理以及进行了简单的实现，在实际真实开发中，首先我们不建议将用户信息返回生成cookie字符串再返回给客户端，因为这是非常隐私的信息，其次要知道cookie是可以直接在客户端更改的，因此鉴权关键字段也是需要斟酌的，现实开发是非常严谨的，请读者在实际使用时秉承严谨的态度。

## JWT鉴权

#### 适用情况

上文已经说到了，session会受到跨域的影响，因此在前后端分离开发以及存在跨域的情况下，我们推荐使用JWT鉴权。

#### JWT鉴权原理

JWT原理和Session大致相同，不同的点在于，JWT生成的Token字符串需要客户端手动存储在**localStorage**或**sessionStorage**中。再次请求时，客户端需要将Token放在请求头的Authorization字段中。
![image-20220705083406574](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220705083406574.png)

#### JWT

jwt是Json Web Token的缩写，它的结构分为三个部分：**header.payload.signature**，两两之间用【.】分隔。

![](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220705103215732.png)

##### header

header是一个JSON结构，主要包含token的类型(即JWT)，签名的算法

```json
{
    "alg":"HS256",
    "typ":"JWT"
}
```

##### payload

payload也是JSON结构，它是存放有效信息的地方，JWT官方提供了一些官方字段，你也可以定义自己的私有字段，其中官方字段如下：

1. iss:签发人
2. exp:token过期时间
3. sub:主题
4. aud:受众
5. nbf:生效时间
6. iat:签发时间
7. jti:编号

但是注意，payload是默认不加密的，因此建议自己定义的私有字段不要放入用户私密信息。

##### signature

它是用户自己定义的字段，用户要设计一个独一无二且保证不会外泄的密钥，通过下方算法生成签名，用于未来的身份验证。

```js
 HMACSHA256(
 base64UrlEncode(header) + "." +
 base64UrlEncode(payload),
 secret)
```

#### 实践

首先安装必要的npm包，执行以下指令：`npm i body-parser cors express express-jwt jsonwebtoken`,在index.js中写入以下内容：

```javascript
// 导入 express 模块
const express = require('express')
// 创建 express 的服务器实例
const app = express()

// 01：安装并导入 JWT 相关的两个包，分别是 jsonwebtoken 和 express-jwt
const jwt = require('jsonwebtoken')
const expressJWT = require('express-jwt')

// 允许跨域资源共享
const cors = require('cors')
app.use(cors())

// 解析 post 表单数据的中间件
const bodyParser = require('body-parser')
app.use(bodyParser.urlencoded({ extended: false }))

// 02：定义 secret 密钥，建议将密钥命名为 secretKey
const secretKey = 'heyyyyfx'

// 04：注册将 JWT 字符串解析还原成 JSON 对象的中间件
// 注意：只要配置成功了 express-jwt 这个中间件，就可以把解析出来的用户信息，挂载到 req.user 属性上
// unless指定哪些接口不需要访问权限，即白名单。
app.use(expressJWT({ secret: secretKey }).unless({ path: [/^\/api\//] }))

// 登录接口
app.post('/api/login', function (req, res) {
  // 将 req.body 请求体中的数据，转存为 userinfo 常量
  const userinfo = req.body
  // 登录失败
  if (userinfo.username !== 'admin' || userinfo.password !== '000000') {
    return res.send({
      status: 400,
      message: '登录失败！',
    })
  }
  // 登录成功
  // 03：在登录成功之后，调用 jwt.sign() 方法生成 JWT 字符串。并通过 token 属性发送给客户端
  // 参数1：用户的信息对象
  // 参数2：加密的秘钥
  // 参数3：配置对象，可以配置当前 token 的有效期，本处设置的是30S
  // 记住：千万不要把密码加密到 token 字符中
  const tokenStr = jwt.sign({ username: userinfo.username }, secretKey, { expiresIn: '30s' })
  res.send({
    status: 200,
    message: '登录成功！',
    token: tokenStr, // 要发送给客户端的 token 字符串
  })
})

// 这是一个有权限的 API 接口
app.get('/admin/getinfo', function (req, res) {
  // 05：使用 req.user 获取用户信息，并使用 data 属性将用户信息发送给客户端
  console.log(req.user)
  res.send({
    status: 200,
    message: '获取用户信息成功！',
    data: req.user, // 要发送给客户端的用户信息
  })
})

// 06：使用全局错误处理中间件，捕获解析 JWT 失败后产生的错误
app.use((err, req, res, next) => {
  // 这次错误是由 token 解析失败导致的
  if (err.name === 'UnauthorizedError') {
    return res.send({
      status: 401,
      message: '无效的token',
    })
  }
  res.send({
    status: 500,
    message: '未知的错误',
  })
})

// 调用 app.listen 方法，指定端口号并启动web服务器
app.listen(8888, function () {
  console.log('Express server running at http://127.0.0.1:8888')
})

```

 开启node服务后，用postman进行测试，调用登录接口后，拿到返回的Token:
![image-20220705102243328](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220705102243328.png)

随后调用获取用户信息接口，注意，该接口是需要权限的，我们需要在请求头中加入**Authorization**字段，值为Token，同时有个注意事项，Token值前需要加入Bearer关键字，用空格分隔，这是必要的操作。
![](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220705102533676.png)

如果你操作的过慢，就会看到如下报错，这是因为我们的有效期只有30s。
![image-20220705102709363](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220705102709363.png)

不妨再调用一次登录接口，同时迅速用新的Token请求用户信息接口，结果如下，说明成功。
![image-20220705102804128](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220705102804128.png)

#### 想说的

##### Token有效期问题

在本文中，我们是自己为Token设置了30s的有效期，但如果你用心观察国内外的网站，貌似没有出现用着用着就突然返回到登录界面让你突然重新登陆的，难道是因为他们的有效期设置的特别长？其实在真实开发中，Token的有效期往往不会用这种方式设置，大多数有效期是动态的，打个比方，只有当你在当前页面半小时之内没有任何请求之后，才会让你的Token自动失效，这种是怎样实现的？其实有很多种实现方案，笔者在此只举一种例子，读者可以先了解一下redis数据库。

##### redis数据库及动态Token解决方案

redis的优点在此不做过多说明，感兴趣的可以自行查阅，redis数据库提供了一个叫**expire**的命令，命令用于设置 key 的过期时间，key 过期后将不再可用。单位以秒计。我们可以以此为基础，当用户请求登录接口时，我们将Token返回给用户，同时我们将这个Token作为Key存储到数据库，Value为这个用户的个人信息或其他内容，并为这个key设置一个定时删除命令，当用户在有效期时，数据库将用户请求接口时携带的Token进行查询，看是否存在这个Token的key，当可以被查询时，说明有效期还在(因为过了有效期这个Token就会被删除，表中就无法查询到这个Token)，同时再次对这个Key执行定时删除任务，达到覆盖上一次删除定时任务，延长有效期的作用，只有当没有接口请求后，删除任务执行，Token才会失效，以此来实现动态Token的目的，至于覆盖定时删除任务这个操作，因为是每一个操作相关的接口都要进行，因此不妨将它封装成全局中间件，避免在每个接口中都写下重复代码。

## 最后

本文所有内容都是基于node的鉴权，相比于纯后端Java开发肯定会有很多不足之处，对于前端而言只是和大家一起了解学习鉴权相关知识，对于文章中如果有任何漏洞还请大家多多指教。