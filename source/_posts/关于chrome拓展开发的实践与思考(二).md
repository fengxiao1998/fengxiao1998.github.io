---
title: 关于chrome拓展开发的实践与思考(二)
date: 2022-05-9 09:03:18
tags: [chrome插件,chrome拓展]
---

## 前言：

上文中，我们已经基于最小的功能实现了chrome拓展插件。接下来我们再为这个插件添加点更复杂的功能。

## 实践：

如图，我们想实现该功能，放置一个button，点击后数字0开始自动加一

![image-20220509093847559](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220509093847559.png)

html文件：

```html
<!DOCTYPE html>
<html>

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <link rel="stylesheet" type="text/css" href="styles/popup.css" />
</head>

<body>
  <main class="popup-main">
    <div class="popup-avatar">
      <img class="popup-image" src="image/avatar.png" alt="">
    </div>
    <div class="popup-information">
      <p class="popup-text">0</p>
      <button class="popup-button" id="popup-button">点击后数量加一</button>
    </div>
  </main>
  <script type="text/javascript" src="js/popup.js"></script>
</body>

</html>
```

js文件：

```javascript
let number = 0
function clickMyButton(){
  number++
  const popupButton = document.querySelector(".popup-text")
  popupButton.innerHTML = number
  console.log('我数量加了1')
}

document.getElementById("popup-button").addEventListener("click", clickMyButton);
```

**tips:**

1. 对于js代码，需要以引入的形式注入，如果直接以script标签包裹js放置到html文件中，会无法运行。感兴趣的可以自己尝试一下。
2. 对于事件绑定，给button标签直接加onclick事件绑定函数也无法正常运行，需要用addEventListener的形式绑定。

------

​		完成以上步骤后，我们更新拓展，并单击右键选择检查将拓展的控制台打开，多次点击按钮，发现数量每次加一，并在控制台出现了打印信息。再次重申，该控制台非按下F12出现的控制台，而是右键单击你的拓展插件在【检查】选项中出现的控制台。

![image-20220509103437659](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220509103437659.png)

 		如果你细心观察，你会发现每次拓展关闭重启后(指再次点击右上角拓展icon)，数字总是回到了0，并且你的拓展控制台也消失掉了。这就是笔者要引入的下一个内容，生命周期与background.js。

**生命周期：**拓展弹出的界面的完整周期只有它展示在界面的那段时间(有点废话)，也就是说，拓展在关闭重新打开后，它的函数也跟着重新刷新，页面也重新渲染，这也就是重新打开后数字变回了0以及控制台自动消失的原因，对于这点你也可以类比我们的chrome浏览器。当浏览器关闭时，控制台也会随之消失。那有没有即使插件关闭了，数字依旧不归0的方法？有的，background.js。

**background.js:**你可以理解为会一直常驻的后台JS或后台页面，它的生命周期与拓展的【default_popup】(即拓展弹出页面)不同，只有当浏览器彻底关闭时，它的生命周期才会结束，我们可以利用该特性完成我们的需求。

------

manifest：

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
  },
  "background": {
    "scripts": ["js/background.js"]//注意引用路径
  }
}
```

在background.js中，写入

```javascript
var a=0
```

在popup.js中，写入

```javascript
//chrome提供的api，该api功能为获取background.js的window
const bg = chrome.extension.getBackgroundPage();
// 点击事件
function clickMyButton() {
  console.log(bg)
  const popupButton = document.querySelector(".popup-text")
  bg.a = bg.a + 1
  popupButton.innerHTML = bg.a
}
document.getElementById("popup-button").addEventListener("click", clickMyButton);
```

**tips:**笔者在函数中的console.log(bg)并非毫无意识，感兴趣的可以在控制台查看这个【bg】到底是何内容，打印后会发现是window对象，并且我们的a变量也在其中。

![image-20220509110000681](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220509110000681.png)

不妨再将background.js的var替换成let，再次打印查看window。

![image-20220509110146598](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220509110146598.png)

​		你会发现window下并没有a变量。当然这是顶层对象和全局变量的知识，与本内容并无太大关系。笔者只是想提醒读者注意变量生命方式避免出现无法获取到变量的低级错误。

​		至此，我们的功能已经完成，不妨来试一试看看是否成功。我们点击按钮将数量变为3，随后刷新浏览器页面，再次打开拓展，发现数字还是0，但只点击一次按钮，会发现数字直接变成了4，这是因为我们是在html文件中给p标签内容填充为0，点击按钮时才会将a的值赋给p标签，这也从侧面验证了html文件的生命周期background.js的生命周期是不同的。关闭浏览器再次打开，会发现一切回到了起点，数字又从1开始进行了，这也验证了background的生命周期。

