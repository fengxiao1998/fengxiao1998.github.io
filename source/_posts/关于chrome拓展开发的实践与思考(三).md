---
title: 关于chrome拓展开发的实践与思考(三)
date: 2022-05-10 11:08:18
tags: [chrome插件,chrome拓展]
---

## 前言：

上文中我们简单阐述了background和拓展的生命周期，接下来我们简单谈谈content-script

## content-script：

你可以理解为注入浏览器页面的js或css文件，我们可设置注入的js和css适用于哪些网站，也可以设置注入的js与css的执行实际等。

#### css注入：

借用之前看到过的文章的段子，给大家讲解下这东西的作用，说有个无礼的客户，就是不喜欢百度页面【百度一下】按钮的颜色，想把这个东西改成其他颜色。

![image-20220510111419279](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220510111419279.png)

但是这玩意是百度的网站啊，研发总不能托人找关系给百度产品打电话说：你们要不换个颜色？

![img](https://gimg2.baidu.com/image_search/src=http%3A%2F%2F5b0988e595225.cdn.sohucs.com%2Fq_70%2Cc_zoom%2Cw_640%2Fimages%2F20200115%2F06c65502495849619a85851c00ba2ab9.jpeg&refer=http%3A%2F%2F5b0988e595225.cdn.sohucs.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1654744507&t=1134c5c3b3ab843914bfce4d7c33c465)

这时候有个大哥就想出了用chrome拓展插件完成这个需求，用的就是"content_scripts"注入。

```json
  "content_scripts": [
    {
      "matches": ["<all_urls>"],
       //匹配地址，指注入js或css的匹配字段，<all_urls>指全匹配，支持正则。
      "js": ["js/content-script.js"],
       //往页面注入的js，注意，注入的js与原页面自带的js是相互隔离的，两个js文件不会相互影响
      "css": ["/styles/content-script.css"],
       //往页面注入的css，注意，css文件是会相互影响的，所以一定要小心使用。
      "run_at": "document_start"
       //注入的js和css的执行时机，可选择参数："document_start", "document_end","document_idle"，最后一个表示页面空闲时，默认document_idle。
    }
  ]
```

接下来我们替“大哥”完成这个离谱的需求，在style文件夹新建content-script.css文件。来到baidu.com。打开元素审查，发现按钮颜色是input标签的background-color。

![image-20220510132015976](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220510132015976.png)

上文已经说过，css的注入会相互影响，我们将利用这个特性，在css文件中写入

```css
#head_wrapper .s_btn{
  background-color: red!important; /*防止颜色优先级不高，加个!important，反正也是图一乐*/
}
```

更新插件并刷新页面后就可以看到，客户的需求完成了。

![image-20220510132402361](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220510132402361.png)

这就是简单的css注入，那我们再来看看js的注入。

#### js注入：

还是以百度为例，来实现个简单的需求，按下【百度一下】按钮时，弹出alert内容，并在localStorage中存入些信息。

content-script.js代码如下：

```javascript
const inputButton = document.querySelector('#head_wrapper .s_btn')
inputButton.addEventListener('click', function () {
  window.localStorage.setItem('baidu','clickButton')
  alert('我按下了按钮')
})
console.log(inputButton)
```

理论上，现在点击按钮时，此时我们会弹出alert，并在localStorage中看到我们的存储信息。但如果读者真正实践时，你会发现功能并没生效，并在控制台发现以下信息：

![image-20220510141934358](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220510141934358.png)

如果仔细查看报错信息，你会发现该段报错表示大致的是【不能将addEventListener绑定在空内容】。有经验的笔者会知道造成该报错的原因是没找到dom元素，这是为何？不妨回头看看我们的manifest文件中content_scripts的run_at选项，是document_start。相信此时你已经明白了为何，该参数表示js文件加载在dom生成之前执行，自然就无法获取到dom节点，所以我们只需将document_start更改为document_end。再次实验，发现点击按钮后弹出alert，查看localStorage，存储的信息也放在其中，至此，我们的功能完成了。

![image-20220510142900595](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220510142900595.png)

