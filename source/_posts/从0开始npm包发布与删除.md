---
title: 从0开始npm包发布与删除
date: 2022-06-15 17:05:18
tags: [npm,node]
description: 从0开始封装、发布、调试、删除一个npm包。
---

## 前言

最近在学习node相关知识，npm自然是与node不可分割的，因此尝试从0封装发布一个npm包，本流程默认你已经对前端有基本的掌握以及安装了node。



## 文件创建与函数封装

#### 方法封装

本次只是想体验封装和发布的过程，因此在功能上不做深入开发，暂定包含一个**监听浏览器关闭事件**功能以及一个**返回数字绝对值**功能。流程如下：

1. 新建空文件夹，注意名字不要包含中文及特殊字符，格式最好使用中线链接，不要包含大写字母，实例：

   ![image-20220615153442513](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615153442513.png)

2. 在空文件夹中打开CMD，输入 `npm init -y`,理论上你就会看到你创建的文件夹下有了一个名为**package.json**的文件，打开文件可以看到内容大致如下：

   ![image-20220615154113047](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615154113047.png)

3. 在**package.json**文件同级新建**READEME.md**以及**index.js**两个文件，**README.md**是我们包的说明书，包含安装方式，提供的方法说明等等，**index.js**即为我们的入口文件，我们封装的方法将在这里创建。至此文件目录如下：

   <img src="https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615154940655.png" alt="image-20220615154940655" style="zoom: 200%;" />

4. 接下来开始封装函数，实现格式化时间功能和返回数字绝对值的功能，在**index.js**文件中写下代码：

   ```javascript
   // 定义监听浏览器关闭事件
   function watchingUnload(fn) {
       window.addEventListener("beforeunload", function () {
           fn()
       })
   }
   
   //定义返回数字绝对值
   function returnAbsNum(number) {
     return Math.abs(number)
   }
   
   module.exports = {
     dateFormat,
     returnAbsNum
   }
   
   // 以上两个功能只是进行最基础的功能封装，在严谨性上有所缺陷。
   ```
   
   在此简单介绍下beforeunload事件，当浏览器整个页面关闭时，会在关闭前触发`beforeunload`事件以及`unload`事件，通常可以在`beforeunload`事件回调中加入一些你想要执行的操作，该事件的兼容性如下，（安利兼容性查看网站：https://caniuse.com/)
   
   **注**：手动刷新页面也会触发该事件。
   
   ![](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/20230530162430.png)
   
5. 在README.md中写入以下内容：

   ~~~markdown
   ## 安装
   ```
   npm install 你包的名字
   ```
   
   ## 导入
   ```js
   const fxTestTool = require('你包的名字')
   ```
   
   ## 监听浏览器关闭事件
   ```js
   fxTestTool.watchingUnload(yourFun)
   ```
   
   ## 返回数字绝对值
   ```js
   //使用htmlEscape方法
   console.log(fxTestTool.returnAbsNum(-100);
   //结果 100
   ```
   
   ## 开源协议
   ISC
   ~~~

至此，npm包开发完成。

#### 细节优化

以上是最基础的包的封装，有很多值得优化的细节，比如，**index.js**文件中不应在这里进行函数的编写与封装，正确的做法应是在根目录下新建一个名为**src**的文件夹，在文件夹中根据方法的不同类型创建对应的js文件，**index.js**只应是在文件中起到引入抛出的作用。优化后目录结构应如下：

<img src="https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615161618855.png" alt="image-20220615161618855" style="zoom:200%;" />

**index.js**中代码如下：

```javascript
const date = require('./scr/dateFormat')

const watchingUnload = require('./scr/watchingUnload')


module.exports = {

 ...date,//当上面两个文件都抛出了很多方法时，解构的写法可以将所有的方法直接抛出而不是需要打点调用

 ...watchingUnload

}
```

不过不进行优化也无所谓，本次的重点并不在此，笔者只是希望大家养成良好的开发习惯。



## npm打包上传

#### npm注册与登录

前期的封装工作已经完成，接下来我们进行发布，如果你没有npm账号，需要先注册一个账号，官网地址https://www.npmjs.com/。在此对注册流程不做过多陈述，注册完成后回到我们的新建文件夹根目录下，在CMD中输入`npm login`,你会看到下方弹出信息，需要你输入用户名，密码，邮箱地址，邮箱验证码。其中注意密码是盲打，因为需要保护你的个人信息，因此盲打下去按回车就好了，一路到底后登录成功。

#### npm打包

登陆成功后，在你要发布的包的根目录输入`npm publish`,稍等片刻，你如果看到了类似以下信息，说明发布成功：

![image-20220615164749263](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615164749263.png)

至此你的npm包发布完成，你可以在你的npm账户中看到你的项目，同时你可以在首页搜索到你的npm包。<img src="https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615164912033.png" alt="image-20220615164912033" style="zoom: 25%;" /><img src="https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615164956399.png" alt="image-20220615164956399" style="zoom:25%;" />



## 安装自己的npm包并进行实验

#### 一、纯js文件演示

我们已经成功发布了自己的包，不仿反向安装一个自己的包试试我们封装的方法，新建一个名为test-my-tools的文件夹，文件夹中新建index.js文件，随后输入 `npm i 你包的名字`，注意，包的名字是你package.json文件中name属性的名字。安装完成后，在index.js中输入以下代码：

```javascript
const fxTestTool = require('你包的名字')

//测试返回绝对值函数
const absNumber = fxTestTool.returnAbsNum(-18)
console.log(absNumber)

```

在该文件夹CMD中输入: `node .\index.js`，打印如下：

![image-20220615172455538](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615172455538.png)

实验成功。

#### 二、VUE文件演示

在vue页面中，通过`import`引入npm包并进行尝试，下文只写入关键性代码

```javascript
import { watchingUnload } from "heyyyyfx-tools";

mounted() {
    watchingUnload(this.testFetch)
    // watchingUnload(this.ajaxFun)
},
methods: {
	async ajaxFun() {
	  try {
	    const response = await axios.get('http://127.0.0.1:3000/users');
	    console.log('查看调用结果', response)
	  } catch (error) {
	    console.error(error);
	  }
	},
	testFetch() {
	  fetch('http://127.0.0.1:3000/users', {
	    method: 'GET',
	    keepalive: true
	  });
	},
  }    
```

​		可以看到，上述代码中，通过`import`引入了`watchingUnload`方法，并在`mouted`中挂载了函数，在`method`中分别写入了两种方法，二者都是通过GET请求调用同一个接口，区别为一个是通过axios请求，另一个是通过fetch请求，在此简单介绍下fetch功能：
​	fetch是JavaScript中用于发送网络请求的API。它是一种基于Promise的API，可以用于发送各种类型的HTTP请求，包括GET、POST、PUT、DELETE等。fetch API使用起来比XMLHttpRequest更加简单，也更加灵活，可以轻松地处理JSON、文件上传等复杂的网络请求。其中有一个名为`keepalive`的属性尤为重要，keepalive属性用于页面卸载时，告诉浏览器在后台保持连接，继续发送数据。
另外***http://127.0.0.1:3000/users***是笔者通过node在本地开启的后端服务并通过cors解决了跨域问题，当接口被调用时，会在服务端打印信息(主要作为验证使用，不开启也无妨)

![](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/20230530163008.png)

准备工作完成后，开启vue服务，打开页面后关闭页面，发现接口请求成功，node端后台成功打印信息。

![](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/20230530164537.png)

说明以下几点：

1. 在vue中成功使用npm包。
2. beforeunload事件成功触发。
3. fetch成功利用keepalive属性在浏览器销毁后继续调用本地接口。

感兴趣的不妨分别改动`keepalive`属性为`false`，或者在事件中选择axios为回调函数看看接口是否被成功执行。通过上述验证，间接实现了一些特定需求，例如：根据用户浏览页面时间长短对用户进行用户画像描写，判断个人喜好等等。

## npm包的删除

我们已经将走完了全部的开发上传实验流程，由于我们的包是测试用包，出于责任感和道德感，应共同维护npm社区，避免上传无用的npm包占用环境和资源，因此现在我们将npm包下架，在CMD中输入`npm  unpublish 你的包的名字 --force` ，完成后会提示以下内容：

![image-20220615180231725](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615180231725.png)

下架成功，至此走完全部流程。
**注意**：删除命令只能删除72小时以内发布的包，删除的包24小时以内不允许重复发布。

## 经验与坑点总结

#### 功能封装的注意事项、可能遇到的问题及解决方案

1. 输入 `npm init -y`时报错：查看文件夹命名是否规范，**不能**包含中文以及特殊字符，用中线命名法。
2. package.json中name需要为英文小写，中线链接，不建议使用大写字母。

#### 上传npm包以及使用npm包

1. npm包的名字具有唯一性，因此请上传包前确认你起的名字是唯一的，可以先在npm官网搜索你要起的名字，如果没有搜索到，你就可以使用这个名字，切记试验结束后及时删除npm包，营造更好的环境。

2. 笔者在下载自己的包时，在空文件夹使用`npm i 包的名字` 命令时，发生了以下报错：

   ![image-20220615181251216](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220615181251216.png)
   看报错信息，笔者一开始以为是全局包出了问题，出现了命名不规范的问题，但在排查后发现并无此包，查询了多种方案后也未能解决，后来笔者在该文件夹下执行`npm init -y`新建了package.json后，再次下载就成功了，目前具体原因未知，后续填坑。

3. `npm i 自己包`未找到，不妨看看是不是自己的npm指向源指向的是淘宝镜像，因为淘宝镜像是隔一段时间进行一次npm官网镜像拷贝，可能你刚上传的npm还未被淘宝镜像拷贝。因此还需要指向npm官网服务器地址。

以上是全部的npm包开发上线下线流程，希望对你有所帮助。