---
title: vue知识
date: 2020-06-30 15:03:18
tags: vue
---

# Node.Js

[官网](http://nodejs.cn/)

#### （扩展了Js的功能，原来在浏览器中运行不了的，可以在node这个平台来运行，具体是v8引擎）

#### 特点：1.异步IO  2.事件驱动  3.事件循环  4.单线程(主线程是单线程，由V8引擎执行)

## 一、整体架构分为三大部分

Native Modules

1. 当前层内容由JS实现
2. 提供应用程序可以直接调用的库，例如fs、path、http、url等模块
3. 但是JS无法直接操作底层硬件设置，所以需要一个桥梁Builtin Modules桥梁

Builtin Modules

1. 在这个桥梁模块中就可以让nodejs核心模块获取到具体的服务支持，从而完成更底层的操作。例如文件的读写行为

v8引擎、libuv库、http、parser等模块

1. v8引擎负责执行js代码、提供桥梁接口（用来实现模块和底层具体实现直接的沟通）
2. libuv库中有事件循环、事件队列、异步IO
3. 第三方模块：http、c-ares等

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e54Lq3EzZX9Ml7Ed/img/f1a4e803-183c-4fd6-a0f7-fe15c307d8c5.png)



## 二、 NodeJs的异步IO

##### libuv库：（相当于一个线程池、可以看作几种不同的异步IO实现方式的封装层）

​		当我们在node中运行了一段js代码后，最终会走到libuv库中，然后它会对当前平台判断，依据平台调用相应的异步IO处理方法

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e54Lq3EzZX9Ml7Ed/img/63d368d7-b415-4b91-83cb-c51a5d8fbdfe.png)



​		Node当中具体实现异步IO的过程：当遇到异步请求，然后libuv库会进行工作，在其内部存在一个事件循环的机制，对异步请求进行管理，

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e54Lq3EzZX9Ml7Ed/img/eecffff4-ba39-44ce-a460-cfa060d3f549.png)



IO是应用程序的瓶颈所在

异步IO提高性能无需原地等待结果返回

IO操作属于操作系统级别，libuv库对其作了封装

NodeJs单线程配合事件驱动以及libuv实现了异步IO

当队列中没有等待的任务时候，它就会退出循环

## 三、事件驱动

1. 特征：主体发布消息，其他实例接受消息
2. （发布者广播消息，其他的订阅者可以监听到之前所订阅的消息，从而在订阅的事件发生之后再去执行相应的处理程序）

​	2. 特点：因为NodeJs拥有一套单线程下异步非阻塞的IO机制，因此程序代码在执行过程中，业务层拿到的并不是最终的目标数据，等到同步代码执行完成之后，此时libuv库就会开始工作,只不过也存在很多非IO的异步操作，比如settimeout，当libuv库接收到一个异步操作请求后，多路分解器开始工作，首先它会找到当前平台下可用的IO处理接口，然后等待着IO操作结束后，将相应的事件通过事件循环或者其他方式添加到事件队列当中，然后最后按照一定的顺序从事件队列中取出相应的事件，然后交给主线程进行执行，（ 其中，事件驱动的体现就是有人发布了事件，然后订阅这个事件的人在将来去接收到具体的事件消息发布之后，就会去执行订阅池的所注册的处理程序 ）

​	3. 顺序：同步代码先执行，等同步代码执行完成后，去执行异步操作，此时会按照注册顺序执行相应的事件处理函数

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e54Lq3EzZX9Ml7Ed/img/206a109c-9615-4ad7-ba88-790f12843883.png) 

## 四、单线程实现高并发

​	通过异步非阻塞IO、事件循环加上事件驱动的架构去通过回调通知的方式来实现非阻塞的调用以及做到并发

​	具体表现为：当遇到多个请求的时候，是无需阻塞的，会由上向下去执行，然后呢去等待着libuv库完成工作之后呢，再按照顺序来通知相应的事件回调去触发							执行就可以了，这样就完成了多线程的工作

## 五、全局对象和全局变量

##### 1. 全局对象

global

##### 2. 全局变量

__filename  ->返回正在执行脚本文件的绝对路径

__dirname  ->返回正在执行脚本所在的目录

timer类函数 ->(settimeout、setinterval)

process    ->提供与当前进程互动的接口

require    ->模块加载

module、exports ->模块的导出

Buffer

###### process（无需require）

process.cwd()  工作目录

process.version node版本

process.arch 操作系统位数

process.env 环境变量

process.env.NODE_ENV

process.env.PATH 系统环境变量

process.platform 操作系统平台

process.argv  启动参数 （返回数组。但是从数组的第二个下标开始才是输入的数据）

process.uptime() 从运行到结束整体消耗时间

process.stdout 标准输出 （返回对象，是一个流）

​	fs.createReadSteam('text.txt').pipe(process.stdout) 输出文件内容

process.stdin 标准输入 （返回对象，是一个流）

  process.stdin.read()

​	process.stdin.pipe(process.stdout)

process.stdin.setEncoding('utf-8')

## 六、常用（核心）模块

### 1.path

basename(path)  获取基础路径，路径的最后部分

dirname(path)   获取路径目录名(路径)

extname

isAbsolute

parse  （得到对象）

format  （对parse后的结果进行序列化）

normalize (**当含有多个杠或者点的时候可以用这个来处理一下)**

##### join（只是简单的拼接，遇到../就按照正常返回上一次进行拼接）

```
path.join('a', 'b', 'c')   // 输出结果为： '/a/b/c'
path.join('a', '/b', 'c')  // 输出结果为： '/a/b/c'
path.join('a/b', '../', 'c') // 输出结果为： '/a/c'
path.join('a', './', 'c') // 输出结果为：'/a/c'
```

##### resolve（从后向前，若字符以 / 开头，不会拼接到前面的路径；若以 ../ 开头，拼接前面的路径，但是不含前面一节的最后一层路径；若以 ./ 开头 或者没有符号 则拼接前面路径；）

```
// 假设当前绝对路径为/admin/user
path.resolve('a', 'b', 'c') // 输出结果为：'/admin/user/a/a/c'
path.resolve('a', '/b', 'c') // 输出结果为： '/b/c'
path.resolve('a/b', '../', 'c') // 输出结果为：'/admin/user/a/c'
path.resolve('a', './', 'c') // 输出结果为：'admin/user/a/c'
```

### 2.Buffer

#### (1) 特点

Node.js中多出来几个概念，一个就是二进制数据、流操作以及Buffer，Buffer就是一片内存空间

1. 无需require的一个全局变量
2. 实现nodeJs平台下的二进制操作
3. 不占据v8的内存空间
4. 内存使用由node来控制，由V8的GC回收
5. 一般配合Steam流使用，充当数据缓冲区

#### (2) 实例对象

​	Buffer类的实例似于整数数组，不过是对应V8堆外的固定大小的原始内存进行分配(堆外内存，不受V8引擎控制)。 缓冲区的大小在创建时已经确定，所以不能调整大小。

创建实例方法

alloc 创建指定字节大小的buffer

```
const b1 = Buffer.alloc(10) // <Buffer 00 00 00 00 00 00 00 00 00 00>
```

allocUnsafe 创建指定字节大小的buffer（不安全，可能混有旧数据）

```
const b2 = Buffer.allocUnsafe(10) // <Buffer ff ff ff ff ff ff ff ff 00 00>
```

from 接受数据（这个数据可以是创建好的buffer），创建buffer，此时他会生成一个全新的缓冲区，和原来的缓冲区不冲突，所以修改接受的buffer，并不会影响新生成的buffer

```
const b1 = Buffer.from('中')
console.log(b1); // <Buffer e4 b8 ad>
console.log(b1.toString()); // 中

const b1 = Buffer.alloc(3)
const b2 = Buffer.from(b1)
 
console.log(b1) // <Buffer 00 00 00>
console.log(b2) // <Buffer 00 00 00>
 
// b1 和 b2 是不是共享内存, 
 
b1[0] = 1
console.log(b1) // <Buffer 01 00 00>
console.log(b2) // <Buffer 00 00 00>
```

#### (3) 实例常用方法

fill  使用数据填充buffer，返回填充后的buffer

write 向buffer中写入数据

toString 提取buffer的数据

slice 截取buffer

indexOf(在buffer中查找数据)

copy拷贝buffer的数据

#### (4) Buffer类静态方法

concat 将多个buffer拼接成一个新的buffer

isBuffer判断当前数据是否是buffer

### 3.FS模块

#### ![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e54Lq3EzZX9Ml7Ed/img/8341efac-b34a-4841-a782-f8608b0899a6.png)



文件权限flag操作符常见有

r： 可读

w：可写

s： 同步

- ： 执行

x： 排他

a： 追加

#### (1) 实例常用方法

- fs.open和 fs.close （打开文件和关闭文件）

```
const fs = require('fs')
const path = require('path')
 
// open 
 
fs.open(path.resolve('data.txt'), 'r', (err, fd) => {
    console.log(fd);
})
 

// close
fs.open('data.txt', 'r', (err, fd) => {
    console.log(fd);
    fs.close(fd, err => {
        console.log('关闭成功');
    })
})
```

- access：判断文件或目录是否具有操作权限
- stat：获取目录及文件信息
- mkdir：创建目录

```
// 三、mkdir recursive 递归操作
fs.mkdir('a/b/c', {recursive: true}, (err) => {
    if (!err) {
        console.log('创建成功');
    } else {
        console.log(err);
    }
})
```

- rmdir：删除目录 ({recursive: true}和mkdir一样)
- readdir：读取目录中内容
- unlink：删除指定文件

### 4.Events模块

- 通过 EventEmitter 类实现事件统一管理
- events 与 EventEmitter
- node.js 是基于时间驱动的异步操作架构，内置 events 模块 events 模块提供了 EventEmitter 类 node.js 中很多内置核心模块继承 EventEmitter
- EventEmitter 常见 API
- on：添加当事件被触发时调用的回调函数
- emit：触发事件，按照注册的序同步调用每个事件监听器
- once：添加当事件在注册之后首次被触发时调用的回调函数
- off：移除特定的监听器
- Eventloop事件环
- 从上至下执行所有的同步代码
- 执行过程中将遇到的宏任务与微任务添加至相应的队列
- 同步代码执行完毕后，执行满足条件的微任务回调
- 微任务队列执行完毕后执行所有满足需求的宏任务回调
- 循环事件环操作

浏览器中的eventloop

```
setTimeout(() => {
  console.log('s1');
  Promise.resolve().then(() => {
      console.log('p1');
  })
  Promise.resolve().then(() => {
      console.log('p2');
  })
})

setTimeout(() => {
  console.log('s2');
  Promise.resolve().then(() => {
      console.log('p3');
  })
  Promise.resolve().then(() => {
      console.log('p4');
  })
})

// s1 p1 p2 s2 p3 p4
setTimeout(() => {
  console.log('s1');
  Promise.resolve().then(() => {
      console.log('p2');
  })
  Promise.resolve().then(() => {
      console.log('p3');
  })
})

Promise.resolve().then(() => {
  console.log('p1');
  setTimeout(() => {
      console.log('s2');
  })
  setTimeout(() => {
      console.log('s3');
  })
})
// p1 s1 p2 p3  s2 s3
```

#### node.js中的eventloop

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/e54Lq3EzZX9Ml7Ed/img/3adfb8dd-5380-4056-80cc-7e6620cbdab2.png) 

timers：执行 setTimeout 与 setInterval 回调

pengding callbacks：执行系统操作的回调，例如top udp

idle，prepare：只在系统内部进行使用

poll：执行与 I/O 相关的回调

check：执行 setImmediate 中的回调

close callbacks：执行 close 事件的回调

其中主要关心timers、poll、check这三个事件队列就好

​	Nodejs 完整事件环

- 执行同步代码，将不同的任务添加至相应的队列
- 所有同步代码执行后回去执行满足条件微任务
- 所有微任务代码执行后会执行timer队列中满足的宏任务
- timer 中的所有宏任务执行完成后就会依次切换队列
- 注意：在完成队列的切换之前会先清空微任务代码
- 微任务队列中，nexttick执行顺序是高于promise.resolve的

```
setTimeout(() => {
    console.log('s1');
    Promise.resolve().then(() => {
        console.log('p1');
    })
    Promise.resolve().then(() => {
        console.log('t1');
    })
});
 
Promise.resolve().then(() => {
    console.log('p2');
})
 
console.log('start');
 
setTimeout(() => {
    console.log('s2');   
    Promise.resolve().then(() => {
        console.log('p3');
    })
    Promise.resolve().then(() => {
        console.log('t2');
    })
});
 
console.log('end');
 
// start end p2 s1 p1 t1 s2 p3 t2
```

#### Nodejs与浏览器事件环区别

​	（1）任务队列数不同

​		  浏览器中只有二个任务队列

​			Nodejs 中有6个事件队列

​	（2）Nodejs 微任务执行实际不同

​			二者都会在同步代码执行完毕后执行微任务

​			浏览器平台下每当一个宏任务执行完毕后就清空微任务

​			Nodejs 平台在事件队列切换时会去清空微任务

​	（3）微任务优先级不同

​			浏览器事件环中，微任务存放于事件队列，先进先出

​			Nodejs 中 process.nextTick 先于 promise.then

### 5. Stream流模块

- Readable：可读流，能够实现数据的读取
- Writeable：可写流，能够实现数据的写操作
- Duplex：双工流，即可读又可写
- Tranform：转换流，可读可写，还能实现数据转换

## 七、中间件

```
1、中间件就是一种功能的封装方式，就是封装在程序中处理http请求的功能，
2、中间件是在管道中执行
3、中间件有一个next()函数，如果不调用next函数，请求就在这个中间件中终止了，
4、中间件和路由处理器的参数中都有回调函数，这个函数有2,3,4个参数
        如果有两个参数就是req和res；
        如果有三个参数就是req,res和next
        如果有四个参数就是err，req，res，next
5、如果不调用next ，管道就会终止，不会再有处理器做后续响应，应该向客户端发送一个响应
6、如果调用了next，不应该发送响应到客户端，如果发送了，则后面发送的响应都会被忽略
7、中间件的第一个参数可以是路径，如果忽略则全部都匹配

const middleware2 = (req, res, next) => {
  console.log('middleware2 start')
  new Promise(resolve => {
    setTimeout(() => resolve(), 1000)
  }).then(() => {
    next()
  })
}
```

## 八、http

```
const http = require('http')
const url = require('url')
const path = require('path')
const fs = require('fs')
const mime = require('mime')
 
const server = http.createServer((req, res) => {
    // 1、路径处理
    // req.query  处理get请求
    // req.body   处理post请求
    // req.params 处理/:xxx这样的请求

    let {pathname, query} = url.parse(req.url)
    pathname = decodeURIComponent(pathname)
    let absPath = path.join(__dirname, pathname)
 
    // 2、目标资源状态处理
    fs.stat(absPath, (err, statObj) => {
        if (err) {
            res.statusCode = 404
            res.end('Not Found')
            return
        }
        if (statObj.isFile()) {
            // 此时说明路径对应的目标是一个文件,可以直接读取然后回写
            fs.readFile(absPath, (err, data) => {
                res.setHeader('Content-type', mime.getType(absPath) + ';charset=utf-8')
                res.end(data)
            })
        } else {
            fs.readFile(path.join(absPath, 'index.html'), (err ,data) => {
                res.setHeader('Content-type', mime.getType(absPath) + ';charset=utf-8')
                res.end(data)
            })
        }
    })
})
 
server.listen(1234, () => {
    console.log('server is running');
})
```

项目中有用到了http-proxy来创建代理服务，就是在我们自己的启动服务外层包一层来把接口代理到对应的ip上