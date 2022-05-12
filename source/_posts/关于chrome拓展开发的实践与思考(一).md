---
title: 关于chrome拓展开发的实践与思考(一)
date: 2022-05-6 15:03:18
tags: [chrome插件,chrome拓展]
---

## chrome拓展文档参考:

1. https://developer.chrome.com/docs/extensions/mv3/
2. http://www.kkh86.com/it/chrome-extension-doc/extensions/api_index.html
3. http://blog.haoji.me/chrome-plugin-develop.html#xie-zai-qian-mian
4. http://chrome.cenchy.com/getstarted.html

## 关于chrome拓展：

关于chrome拓展，也有的人称为chrome插件，英文官方名称为Chrome extensions，在此作者将使用拓展称呼，不多做纠结，原生开发中大多数使用是html、json、js、css等文件，个人认为chrome拓展对于前端开发是较为友好。笔者自身就是前端，因此在下方描述时将默认大家都对前端有所研究，提前对不能照顾到的人说声抱歉。

## 实践开发：

​	刨除你的拓展上线发布流程外，完整的开发调试流程如下：

1. 新建文件夹，在文件夹根目录中放置你的manifest.json文件，并围绕该文件放入你自己需要的内容。

2. 完成开发后在chrome浏览器中进入拓展程序![image-20220507172017421](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507172017421.png)

3. 在右上角你可以看到开发者模式开关，将它打开![image-20220507172040129](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507172040129.png)

4. 随后点击左上角出现的【加载已解压的拓展程序】，选择你的拓展程序根目录即可。

5. 随后你可以看到你的拓展程序出现。

   ![image-20220507143022354](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/chrome-extensions2.png)

6. 在浏览器右上角你也可以看的你的拓展icon

   ![image-20220507172058217](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507172058217.png)

   

​		整个结构需要基于名为【manifest】的json文件进行开发，你可以理解为它是我们与chrome的桥梁，chrome通过读取该文件的内容来实现我们拓展的功能与展示，manifest.json的配置单部分内容如下：



```json
{
  "manifest_version": 2,
    //告诉chrome需要用哪个版本来解析我们的文件，目前有且只有2与3，两者区别在于manifest的部分配置写法不同，笔者将在下文中进行举例，在本次实践中使用的是2.
    
  "name": "FxExtensions",//给你的拓展起个名字，可以为中文。
  "description": "it is a description",//对这个拓展进行一些描述。
  "version": "1.0",//你的拓展的版本号，一般只有拓展上线市场时才会用到，自己本地娱乐无需在意。

  "icons": {
    //此为你在拓展程序主页面展示的icon
      "16":'image/icon.png',//可以设置不同尺寸对于不同的icon。
      "48":'image/icon.png',
      "128":'image/icon.png'
  },
    
  "browser_action": {
    // 浏览器右上角图标设置，browser_action、page_action、app必须三选一，前两者区别为，browser为所有网站常驻且常量(icon不为灰色),page为只在特定网站常量，其余时间为灰色(可以理解为不可使用状态),但二者可配置项完全一致，笔者思考是否page_action还有额外的配置项如：配置哪个页面为可使用状态。但查阅官方文档	发现并没有如笔者所想，配置哪个页面为可使用状态需要用写在background.js(下文会对此说明)的chrome.api来进行调用，因此笔者觉得有些不合理性，在manifest_version为3的版本中，browser_action与page_action也确实统一使用单action字段，不再区分二者，对于app，笔者在查阅中发现与前两者相差较大，且使用人数貌不是很多，因此在此小挖一坑，日后来填，感兴趣的可自行查阅。
    "default_icon": "image/icon.png",//此为你在浏览器右上角显示的icon
    "default_popup": "popup.html",//当你点击浏览器右上角显示的icon时(不为灰状态),弹出的展示页面
    "default_title": "鼠标悬浮展示内容"//鼠标悬浮在icon时显示的title文字。与image标签的title属性类似。
  },
    
  "background":{
    //常驻后台的JS或者页面，通常为JS，，该配置如它的名字一样，在背后默默支持，一般会在浏览器开启时开始执行，浏览器关闭后结束，一般用来跑一些程序函数等，在下文中会有详细说明。
		"page": "background.html"//2种指定方式，如果指定JS，那么会自动生成一个背景页
		//"scripts": ["js/background.js"]
	},
    
	"content_scripts":[
      // 需要直接注入页面的JS，该内容会在下文与background一同说明。
		{
			//"matches": ["http://*/*", "https://*/*"],
			// "<all_urls>" 表示匹配所有地址
			"matches": ["<all_urls>"],
			// 多个JS按顺序注入
			"js": ["js/jquery-1.8.3.js", "js/content-script.js"],
			// JS的注入可以随便一点，但是CSS的注意就要千万小心了，因为一不小心就可能影响全局样式
			"css": ["css/custom.css"],
			"run_at": "document_start"
            // 代码注入的时间，可选值： "document_start", "document_end", or "document_idle"，最后一个表示页面空闲时，默认document_idle
		},
	],
    

	"permissions":[
     //权限申请，当你想要使用chrome提供的api能力时，你需要向chrome"报备"，该数组就是报备列表。如果不需要它的能力，就不需要写该配置。
		"contextMenus", // 右键菜单
		"tabs", // 标签
		"notifications", // 通知
		"webRequest", // web请求
		"webRequestBlocking",
		"storage", // 插件本地存储
		"http://*/*", // 可以通过executeScript或者insertCSS访问的网站
		"https://*/*" // 可以通过executeScript或者insertCSS访问的网站
	]
    //除此之外还有很多配置项，在此不一一列举，感兴趣的可自行查阅
    //version3:https://developer.chrome.com/docs/extensions/mv3/manifest
    //version2:https://developer.chrome.com/docs/extensions/mv2/manifest
}
```

接下来我们进行一个最小成本最少配置的开发，manifest.json如下：

```json
{
  "manifest_version": 2,
  "name": "FxExtensions",
  "description": "it is a description",
  "version": "1.0",
  "icons": {
    "16":"image/icon.png",
    "48":"image/icon.png",
    "128":"image/icon.png"
  },
  "browser_action": {
    "default_icon": "image/icon.png",
    "default_popup": "popup.html",
    "default_title": "鼠标悬浮展示内容"
  }
}
```

可以看到，整个配置单只有基本的配置项，不包含background,permissions,content_scripts等参数。在配置单上我们可以看到，配置上指向了两个文件，一个是icon，另一个是html文件，因此我们还需要一个icon.png以及一个popup.html。整个目录结构如下

![image-20220507170441977](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507170441977.png)

出于便于后期维护以及代码的干净整洁，我们将未来可能用到的内容新建文件夹，便于后续管理，实际不使用文件夹一股脑放在根目录也是可以，icon.png放在了image文件夹下，icon大小官方建议大概是19px*19px。接下来我们编辑popup.html的内容，一步一步来，我们先做到基本静态展示，html代码示例如下：

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    *{
      padding: 0;
      margin: 0;
    }
    .popup-main{
      background-color: #ecf5ff;
      padding: 8px;
      height: 150px;
      overflow: hidden;
    }
    .popup-avatar{
      margin: 0 auto;
      width: 100px;
      height: 100px;
      border-radius: 4px;
      overflow: hidden;
    }
    .popup-image{
      width: 100px;
      height: 100px;
    }
    </style>
</head>

<body>
  <main class="popup-main">
    <div class="popup-avatar">
      <img class="popup-image" src="image/avatar.png" alt="">
    </div>
  </main>
</body>

</html>
```

可以看到，整个html的功能是展示了一个名为【avatar】的图片，并在样式上进行了修改。avatar读者可以自行随意选择一个你有的图片放在image文件夹下，笔者在此不做提供。完成整个代码编写后，参照上文提到的打包流程，点击【加载已解压的拓展程序】按钮，选择新建的文件夹，理论上你就会看到你的拓展程序加载成功了。

![image-20220507172115390](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507172115390.png)

在浏览器右上角左键点击你的icon，弹出内容大致如下：

![image-20220507172305210](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507172305210.png)

你也可以右键单击弹出内容，审查元素，即我们常用的F12。

![image-20220509092641646](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220509092641646.png)

如果你成功出现了该步骤，说明目前你的操作是正确的，如果你没能在右上角找到你的icon,不妨点击这里看看是否没有将它常驻在浏览器上。

![image-20220507172445620](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220507172445620.png)

至此，最基本的拓展功能便已经完成。不妨对这个页面进行你的个性化改动，打造属于你的拓展页面。

**tips:** 更改代码后无需重新打包，只需在拓展页面点击左上角【更新】按钮并刷新浏览器即可(有时候这两个步骤也不需要，只需要点击右上角插件icon再次开启关闭即可)