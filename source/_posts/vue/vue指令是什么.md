---
title: vue指令
date: 2020-06-30 15:03:18
tags: vue
---
# 一，vue指令是什么

参考链接(https://cn.vuejs.org/v2/guide/custom-directive.html)

对于vue我们最熟悉的可能是他的组件了，但是他的指令也是项目中作用非常大的一种工具.

vue自带很多指令例如：v-on, v-bind,v-if,v-show,v-for......

指令可以理解是以（v-）开头的语法糖，用来实现部分公共逻辑



# 二，如何创建一个自定义指令

1.全局创建一个指令

```
// 注册一个全局自定义指令 `v-focus`
Vue.directive('focus', {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el) {
    // 聚焦元素
    el.focus()
  }
})
```



2.创建一个私有的指令

组件中接受一个 directives 的选项：

```
directives: {
  focus: {
    // 指令的定义
    inserted: function (el) {
      el.focus()
    }
  }
}
```



# 三，何时应该使用vue指令来解决问题

对于指令其实能做的事情非常多，有时候不用指令当然也能实现，那么问题来了，我们在什么时候应该使用vue的指令去解决问题呢？

我是这样理解的：当一个逻辑需要对dom进行操作的时候，皆建议使用指令来实现，当这种逻辑运用的非常频繁的时候，建议抽样成一个全局指令。



# 四，项目中实际使用的vue指令举例

1.图片懒加载插件

main.js 中使用懒加载插件



```
Vue.use(VueLazyload, {
  preLoad: 1.3,
  error: require('@/assets/default.png'),
  // loading: 'dist/loading.gif',
  try: 3 // default 1
})
```



实际使用该指令



```
<img
  :key="current.src"
  class="file-item image"
  v-if="current.src"
  v-lazy="current.src"
  :alt="current.title"
  v-show="!loading"
>
```



2.自己封装的 vue指令(滚动条滚动到底部时的事件)



```
let handlerFn
export default {
  // 当被绑定的元素插入到 DOM 中时……
  inserted: function (el, binding) {
    let cb = binding.value
    let scrollEl = el
    handlerFn = (event) => {
      // console.log(scrollEl)
      let scrollTop = scrollEl.scrollTop
      let blockHeight = scrollEl.clientHeight
      let scrollHeight = scrollEl.scrollHeight
      if (scrollTop + blockHeight === scrollHeight) {
        console.log('已经到最底部了！!')
        cb && cb()
      }
    }
    scrollEl.addEventListener('scroll', handlerFn)
  },
  unbind (el) {
    let scrollEl = el
    scrollEl.removeEventListener('scroll', handlerFn)
  }
}
```



封装的有点简陋~~



# 五，使用vue指令的优势

总结来说，vue 指令的优势就是，将部分与业务没有太大关系，但是要对dom进行操作 的逻辑 抽离到指令中。在使用时 直接使用v-xx 来调用，简化代码，并且提升代码的维护性。