---
title: 关于chrome拓展开发的实践与思考(四)
date: 2022-05-10 14:40:18
tags: [chrome插件,chrome拓展]
---

## 前言：

目前我们已经在popup.js，background.js，content-script.js中玩转了一番，但我们只是将这三个文件分别进行了实践，真实的开发中，肯定是三份文件共同配合相互协作的，因此，三个文件的相互通信就显得尤为重要。

## 拓展之间的通信：

其实我们在之前的实践中可以了解到，popup.js与background.js都是在拓展的上下文中执行的，因此其实我们可以将它们归为一大类。而content-script.js的执行是在浏览器web页面执行的。因此我们可以总结为，拓展内部通信和拓展外部通信。

#### popup.js与background.js的通信：

对于二者的通信，前文中其实已经有所涉及，我们曾在popup.js中通过`chrome.extension.getBackgroundPage`拿到了background下的window对象，对于background拿到popup.js的信息，用到的方法为：

```javascript
chrome.extension.getViews({type:'popup'})//要获取的视图类型。如果省略，返回所有视图（包括后台页面和标签页）。有效值为："tab"（标签页）、"infobar"（信息栏）、"notification"（通知）、"popup"（弹出窗口）。
```

background.js代码如下:

```javascript
var aa = chrome.extension.getViews({type:'popup'})
//直接执行赋值操作
function getPopup(){
    //在函数中获取，并return
    var views = chrome.extension.getViews({type:'popup'})
    return views
}
```

popup.js代码如下:

```javascript
function getPopup(){
  const bg = chrome.extension.getBackgroundPage();
  console.log('我直接拿bg的window下的aa',bg.aa)
  console.log('我在popup.js页面,通过调用background的函数方法拿到popup的值',bg.getPopup())
}
getPopup()
```

读者可能会好奇为何还要在popup.js中执行background.js中的获取popup的方法(没绕过来的自己再读两遍)，不妨打开元素审查看看你的打印内容。

![image-20220510172237237](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220510172237237.png)

可以发现，同样的方法，aa是空数组，而views拿到了window，这又回到了生命周期的问题，background的生命周期在浏览器打开的时候就执行了，而此时拓展并没有开启，也就是popup的window对象并未生成，自然也就拿不到了，只有开启了popup页面，popup.js执行方法，background中才拿到popup的window对象，读者在实际开发中也应注意该问题。

**tips**:注意，V3版本中getBackgroundPage方法不再适用，笔者在尝试中发现返回了undefined。相互通信建议使用chrome.runtime.sendMessage` & `chrome.runtime.sendMessage。笔者因为一直使用V2版本，结果今天尝试使用V3实践时遇到了很多坑点，且V2版本在2023年即将弃用，建议多看看官方文档。

#### popup.js与content-script的通信：

popup是基于拓展的js文件，content-script是注入页面的js文件，二者的交流可以通过浏览器控制台和拓展控制台清晰的看到，为了方便测试，本着好用就往死里用，我们再次来到百度搜索页面(bushi。

popup：

```javascript
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
  console.log('收到来自content-script的消息：');
  console.log(request, sender);//request即为收到的消息，sender为发送源所在的页面，
  sendResponse('我是后台，我已收到你的消息：' + JSON.stringify(request));//sendResponse为收到消息后的回调函数
});
```

content-script:

```javascript
//理论代码，使用chrome提供的api即可完成信息发送，参数[data]为代称，它可以是Object对象，也可以是数组，字符串等，本处data只做展示，读者可自行更改参数类型。笔者将data更改为数字1
 chrome.runtime.sendMessage(data , function(response) {
      console.log('收到来自后台的回复：' + response);
  });


// 实际代码，因为理论代码只提供理论，如果直接在content-script中放入理论代码，更新插件后，任意打开页面，代码执行，但此时拓展并未开启，也就无法在控制台查看消息与给予反馈，代码确实执行了但无从监听，因此需要手动触发事件。
//'#head_wrapper .s_btn'是百度搜索页面的【百度一下】按钮类名，之所以监听mouseenter而不是click是因为click之后页面会刷新，看不到我们想要的信息。
document.querySelector('#head_wrapper .s_btn').addEventListener('mouseenter',
  function(){
    chrome.runtime.sendMessage(1 , function(response) {
      console.log('收到来自后台的回复：' + response);
    });
  }
)
```

全部完成后，来到百度搜索页面并打开控制台，打开拓展并打开拓展控制台，就绪后鼠标放到【百度一下】按钮。可以看到控制台打印信息，二者完成相互通信。![image-20220511172419321](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220511172419321.png)

#### content-script与background.js的通信：

这两者的通信方式与前两者大致相同，但理论上这两个js都是打开浏览器页面时自动运行，因此理论上我们使用最简代码即可完成通信。

content-script：

```javascript
chrome.runtime.sendMessage({data:'发给background'} , function(response) {
  console.log('收到来自后台的回复：' + response);
});
```

background:

```javascript
chrome.runtime.onMessage.addListener(function(request, sender, sendResponse) {
    console.log('收到来自content-script的消息：');
    console.log(request, sender);
    sendResponse('我是后台，我已收到你的消息：' + JSON.stringify(request));
  });
```

更新拓展，在任意页面(除了扩展程序管理页面)打开浏览器控制台即可看到下面的信息。

![image-20220512094655524](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220512094655524.png)

至此，通信大致全部完成。