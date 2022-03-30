#### **前言**

 

call()、apply()、bind()是Function.prototype上的方法，所有函数都可以调用，它们的第一个参数都是this的指向对象。

 

#### **举栗子：以“小明学会骑自行车”为例。**

 

先声明一个函数，并且执行它

 

代码块1：

 

```
// 学车方法
function myFn (a) {
a.info = '学会了骑车'
console.log('this指向', this)
}
// 叫小明的一个个体
var obj = {
name: '小明',
info: '不会骑车'
}
myFn(obj)
// console.log输出结果为：this指向 Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, parent: Window, …}
```

 

我们可以看到函数里面this 指向是Window

 

#### **call()**

 

现在，我们想要小明通过这个方法学会骑自行车，那我们可以通过call方法来重定义this指向，将this指向小明这个对象,使他快速学会骑车

 

代码块2：

 

```
// 将上述myFn方法用call执行
// 学车方法
function myFn (a) {
a.info = '学会了骑车'
// 输出1
console.log('this指向', this)
// 输出2
console.log(a.name + a.info)
}
// 叫小明的一个个体
var obj = {
name: '小明',
info: '不会骑车'
}
myFn.call(obj,obj)
// 输出1 this指向 {name: "小明", info: "学会了骑车"}
// 输出2 小明学会了骑车

 
```

比如还可以将小明的年龄另外传入

 

代码块3：

 

```
// 先将myFn方法修改下
function myFn (a,b) {
a.info = '学会了骑车'
// 输出
console.log(a.name + '在' + b + '岁' + a.info)
}
// 叫小明的一个个体
var obj = {
name: '小明',
info: '不会骑车'
}
myFn.call(obj,obj,'10')
// 输出 小明在10岁学会了骑车

 
```

上述结果，我们可以知道，call方法可以将调用函数执行，并将this 指向第一个参数，后续其他参数将会全部传递给调用函数使用

 

**同样也可以使用apply和bind来实现**

 

#### **apply()**

 

延续代码块3，我们根据官方介绍使用apply方法

 

```
// 只需将最后的调用方式变一下
myFn.apply(obj,[obj,'10'])
// 输出 小明在10岁学会了骑车
```

 

可见 call()方法的作用和 apply() 方法类似，区别就是call()方法接受的是参数列表，而apply()方法接受的是一个参数数组。

 

引申：如果想要从[1,2,5,8,3]中找出最大值，我们就可以想到通过apply借用Math对象中的max来求最大值。

 

```
var numbers = [1,2,5,8,3]
var max = Math.max.apply(null, numbers)
console.log(max) // 8
```

 

在非严格模式下，如果call()和apply()的第一个参数是null或者undefined，那么this的指向就是window全局变量

 

#### **bind()**

 

言归正传，使用bind看一下

代码块4：

 

```
var newFn1 = myFn.bind(obj,obj,'10')
// 输出1
console.log(newFn1)
var newFn2 = myFn.bind(obj,[obj,'10'])
// 输出2
console.log(newFn2)
// newFn1、newFn2输出结果
/* myFn (a,b) {
a.info = '学会了骑车'
// 输出
console.log(a.name + '在' + b + '岁' + a.info)
*/
// 执行newFn2
newFn2()
// s输出 小明在10岁学会了骑车
```

 

可见bind返回的是方法，它的参数和 call 一样，所以需要调用才是得到最终结果

 

#### **总结**

 

1.call()、apply()和bind()都是可以用来改变this指向，可借助它们实现继承；

 

2.call()与apply()区别在于参数不一样，如果想一目了然表达形参和实参的对应关系，用call()比较合适；

 

3.bind()是返回一个新函数，可供以后调用，而apply()和call()是立即调用，并返回结果