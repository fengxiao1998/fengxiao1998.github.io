---
title: nginx在window环境下前端说明与实践
date: 2022-05-20 15:03:18
tags: [nginx]
---





## 前言：

本内容是基于**windows前端**进行的简单nginx配置，对于Linux不做过多说明。

## 下载与安装：

nginx官方下载地址:http://nginx.org/en/download.html

任选其一即可。

![image-20220520163809216](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220520163809216.png)

解压后结构如下：

![image-20220520163945687](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220520163945687.png)

## 目录结构与说明：

**注意：**不要用点击nginx.exe的方式启动nginx，后果是无法通过指令**关闭或重启nginx**，会让你怀疑是不是哪里出了问题，改了配置却没生效。如果你眼疾手快已经点开了，不妨在任务管理器中手动kill掉。

![image-20220520164302319](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220520164302319.png)

接下来看下整个目录，其实目前不需要在意太多东西，只打开conf下的nginx.conf文件即可。浏览整个文件结构，不难看出是多层嵌套的格式，结构解释如下：

```
...              #全局块
events {         #events块
   ...
}
http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location    #location块
        {
            ...
        }
        location 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
```

- **全局块**：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
- **events块**：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
- **http块**：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
- **server块**：配置虚拟主机的相关参数，一个http中可以有多个server。
- **location块**：配置请求的路由，以及各种页面的处理情况。

## 启动nginx与修改配置：

目前看着有点难懂，倒也不必在意，不妨直接动手尝试下，在你的nginx根目录下打开cmd(这个应该都会吧？就不放截图了)，在cmd中输入`start nginx`，不出意外的话你应该能看到一个黑色窗口一闪而过，说明启动成功了。浏览器url地址栏输入`locallhost:80`，理论上你就能看到你的nginx页面。

![image-20220520171816873](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220520171816873.png)

再试试输入locallhost:80/50x.html，理论上会看到这个页面。

![image-20220520171908183](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220520171908183.png)

如果你页面成功显示出来了，说明至少目前是一切顺利的，接下来我们就要进行我们自己的配置，让我们把目光直接放在nginx.conf文件下的server代码块，其他的无需改动与关注，默认的server模块大致是这样子的：

```
   server {
        listen       80;#监听端口
        server_name  localhost;#监听地址
        
        location / {
          	root   html;
        	index  index.html index.htm;
        }
        
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
```

看到上面的代码，即使对参数配置一窍不通，你大概也能看到我们刚才输入的url地址在代码中有所出现，监听地址和端口不做说明，root你可以理解为你要展示的页面所在位置，index自然就是首页对应的html文件，顺着这个思路我们可以发现nginx根目录有个html文件夹，文件夹中50x.html和index.html文件就躺在那里。server内的简单结构我们目前就知道了。

不知道大家在vsCode中有没有Live Server这个拓展，没有的话可以在拓展市场安装一个，稍后会用到，接下来你可以将你的root地址换成你自己的html所在位置看看，笔者用的是自己博客的地址 C:\Users\fx\Desktop\myblog;然后输入cmd中输入`nginx -s reload`，刷新页面理论上你就能看到属于你自己的页面。

接下来我们试试其他玩法，随意找个html文件用vscode打开，按下alt+L+O，这是live Server的快捷键，你会看到你的页面被端口为localhost:5500的地址打开了，这是live Server的功劳接下来我们对server模块换个写法：

```
  server {
       listen       8080;
       server_name  localhost;
       charset utf-8;
       location / {
        proxy_pass   http://localhost:5500/;
       }
    }
```

然后输入cmd中输入`nginx -s reload`，url地址输入localhost:8080，你会看到展示内容与localhost:5500一致，此时如果你销毁localhost:5500(比如关闭你用live Sever跑的VScode),nginx代理的页面也会随之无法使用。

**附**：我在很多文章下面看很多人在配置server_name时总会更改为xxxx.com诸如此类，但如果你实际尝试是无法生效的，因为这个的前提是你需要在window的host文件下写入你的网站对应ip地址，这就好比你输入localhost:8080会自动帮你转跳到你的实际ip地址，因为这是需要提前配置的，所以如果你更改了server_name 没生效也不要担心。