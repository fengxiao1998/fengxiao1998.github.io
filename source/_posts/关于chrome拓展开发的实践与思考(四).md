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

