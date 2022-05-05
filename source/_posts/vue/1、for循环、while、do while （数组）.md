---
title: 关于JS循环
date: 2020-06-30 15:03:18
tags:
---
**1、****for循环、while、do while** **（数组）**



能正确响应break、continue和return语句

```
let arr = [1, 2, 3]
//for循环遍历数组
for(let i=0;i<arr.length;i++){
  console.log(i+": "+arr[i])
}
// 输出
 0: 1 
 1: 2
 2: 3

let arr=[1,0,8,7],i=0;
while(i<arr.length){
    console.log(i+':'+arr[i]);//0:1 1:0 2:8 3:7
    i++
}

let arr=[1,0,8,7],i=0;
do{
   console.log(i+':'+arr[i]); //0:1 1:0 2:8 3:7
   i++
}while(i<arr.length)
```



**2、for...in （对象和数组）**   可得到数字索引属性、字母索引属性、原型上的属性，无法获取不可枚举属性

for...in 得到的数字索引属性为 string 类型

for-in循环是为了遍历对象而设计的，事实上for-in也能用来遍历数组，但定义的索引i是字符串类型的。如果数组具有一个可枚举的方法，也会被for-in遍历到

能正确响应break、continue和return语句

```
for(let i in arr){
  console.log(i+": "+arr[i])
}
// 输出
 0: 1 
 1: 2
 2: 3

//for-in会遍历到数组的属性
var arrTmp = ["value1","value2","value3"]
arrTmp.name="myTest";
for(var i in arrTmp){
    console.log(i+":"+arrTmp[i])
}
// 输出
	0:value1
  1:value2
  2:value3
  name:myTest
```



**3、for...of（ES6）（对象和数组）**   只可以得到数字索引的属性值

用来遍历各种类数组集合，例如DOM NodeList对象、Map和Set对象，甚至字符串也行

可以正确响应break、continue和return语句



官方说法：

for...of语句在可迭代对象(包括 Array, Map, Set, String, TypedArray，arguments 对象等等)上创建一个迭代循环，对每个不同属性的属性值,调用一个自定义的有执行语句的迭代挂钩



```
// for-of遍历数组，不带索引，i即为数组元素
for(let i of arrTmp){
    console.log(i)
}
//输出 "value1" "value2" "value3"
 
// for-of遍历Map对象
let iterable = new Map([["a", 1], ["b", 2], ["c", 3]]);
for (let [key, value] of iterable) {
  console.log(value);
}
//输出 1 2 3
 
// for-of遍历字符串
let iterable = "china中国";
for (let value of iterable) {
  console.log(value);
}
```



**4、Object.keys()，Object.values()，Object.entries()**   

 ES6 提供三个新的方法 —— entries()，keys()和values() —— 用于遍历数组。它们都返回一个遍历器对象，可以用for...of循环进行遍历，唯一的区别是keys()是对键名的遍历、values()是对键值的遍历，entries()是对键值对的遍历



Object.entries() 方法返回一个给定对象自身可枚举属性的键值对数组，其排列与使用 for...in循环遍历该对象时返回的顺序一致（区别在于 for-in 循环也枚举原型链中的属性）。通俗点就是 Object.entries() 可以把一个对象的键值以数组的形式遍历出来，结果和 for...in 一致，但不会遍历原型属性。

```
for (let index of ['a', 'b'].keys()) {
console.log(index);
}
// 0
// 1
for (let elem of ['a', 'b'].values()) {
console.log(elem);
}
// 'a'
// 'b'
for (let [index, elem] of ['a', 'b'].entries()) {
console.log(index, elem);
}
// 0 "a"
// 1 "b"

const obj = { foo: 'bar', baz: 'abc' }; 
console.log(Object.entries(obj));  // [['foo', 'bar'], ['baz', 'abc']]

const arr = [1, 2, 3]; 
console.log(Object.entries(arr));  // [['0', 1], ['1', '2'], ['2', '3']]

const arr1 = [{ a: 1 }, 2, 3]; 
console.log(Object.entries(arr1));  // [['0', { a: 1 }], ['1', '2'], ['2', '3']]

const arr2 = [{ a: 1 }, { b: 2 }, { c: 3 }]; 
console.log(Object.entries(arr2));  // [['0', { a: 1 }], ['1', { b: 2 }], ['2', { c: 3 }]]

const str = '123'; 
console.log(Object.entries(str));  // [['0', '1'], ['1', '2'], ['2', '3']]

const num = 123; 
console.log(Object.entries(num));  // []

const float1 = 12.3; 
console.log(Object.entries(float1));  // []

const obj2 = { foo: 'bar', baz: 'abc' }; 
console.log(Object.entries(obj2));  // [['foo', 'bar'], ['baz', 'abc']]
const map = new Map(Object.entries(obj2)); 
console.log(map); // Map {'foo' => 'bar', 'baz' => 'abc'}
```



5、**forEach()（数组）**

forEach只能遍历数组，且无法中断遍历

遍历数组中的每一项，没有返回值，对原数组没有影响，不支持IE

```
var arr = ['a', 'b', 'c', 'd'];
arr.forEach(function(value, index) {
    console.log('value=', value, 'index=', index);
})
// 输出
    value= a index= 0
    value= b index= 1
    value= c index= 2
    value= d index= 3
```



**6、map()**

可以对遍历的每一项做相应的处理,返回每次函数调用的结果组成的数组

```
var arr = ['a', 'b', 'c', 'd'];
arr.map(function(item, index, array) {
    console.log(item, index);
	return item + index
})
// 输出
  a 0
  b 1
  c 2
  d 3

(4) ["a0", "b1", "c2", "d3"]
```



7、**Objcet.getOwnPropertyNames() （对象和数组）**

可以得到数组和对象自身的所有属性，包括不可枚举属性，但是无法获取原型上的属性



```
var obj = { 'name': "yayaya", 'age': '12', 'sex': 'female' };
Object.prototype.pro1 = function() {}
    Object.defineProperty(obj, 'country', {
    Enumerable: true,
    value: 'ccc'
});
Object.defineProperty(obj, 'nation', {
    Enumerable: false //不可枚举
})
obj.contry = 'china';
Object.getOwnPropertyNames(obj).forEach(function(index) {
    console.log(index, obj[index])
});
// 输出
    name yayaya
    age 12
    sex female
    country ccc
    nation undefined
    contry china
```



**8、Reflect.ownKeys() （对象和数组）**

返回对象自身的所有属性,不管属性名是Symbol或字符串,也不管是否可枚举.



```
Object.defineProperty(目标对象，给对象要加的属性);返回加了属性之后的对象
Object.getOwnPropertyNames(目标对象);返回带有该对象属性的数组
Object.is(value1,value2);判断2个值是否相同，返回布尔值

var obj = [1,2]

Reflect.ownKeys(obj)
// 输出
(3) ["0", "1", "length"]


var obj = { 'name': "yayaya", 'age': '12', 'sex': 'female' };
Object.prototype.pro1 = function() {}
Object.defineProperty(obj, 'country', {
    Enumerable: true,
    value: 'ccc'
});
Object.defineProperty(obj, 'nation', {
    Enumerable: false //不可枚举
})
obj.contry = 'china';
Reflect.ownKeys(obj).forEach(function(index) {
    console.log(index, obj[index])
});
// 输出
    name yayaya
    age 12
    sex female
    country ccc
    nation undefined
    contry china
    
    
```



**9、sort（）**

排序

```
arr.sort((a,b)=>a-b)//升序
arr.sort((a,b)=>b-a)//降序
```



10、**some()**

- some作为一个用来检测数组是否满足一些条件的函数存在，同样是可以用作遍历的函数签名同forEach，有区别的是当任一callback返回值匹配为true则会直接返回true，如果所有的callback匹配均为false，则返回false。
- some() 方法会依次执行数组的每个元素：

- 如果有一个元素满足条件，则表达式返回true , 剩余的元素不会再执行检测。如果没有满足条件的元素，则返回false。



```
var arr = [1, 2, 3];
arr.some((item, index, arr) => { // item为数组中的元素，index为下标，arr为目标数组
    console.log(item); // 1, 2, 3
    console.log(index); // 0, 1, 2
    console.log(arr); // [1, 2, 3]  
})
```



11、**every()**

every() 方法用于检测数组所有元素是否都符合指定条件（通过函数提供）。

　　every() 方法使用指定函数检测数组中的所有元素：

- 如果数组中检测到有一个元素不满足，则整个表达式返回 false ，且剩余的元素不会再进行检测。
- 如果所有元素都满足条件，则返回 true。



```
var arr = [1, 2, 3];
arr.every((item, index, arr) => { // item为数组中的元素，index为下标，arr为目标数组
    return item > 0; // true
    // return index == 0; // false
})
```



12、**filter()**

filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。



```
var arr = [1, 2, 3];
arr.filter(item => { // item为数组当前的元素
    return item > 1; // [2, 3]
})
```



13、**reduce()**

reduce() 方法接收一个函数作为累加器（accumulator），数组中的每个值（从左到右）开始缩减，最终为一个值。



```
var total = [0,1,2,3,4].reduce((a, b)=>a + b); //10
```



reduce接受一个函数，函数有四个参数，分别是：上一次的值，当前值，当前值的索引，数组



```
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, index, array){
 return previousValue + currentValue;
}); // 10
```



reduce还有第二个参数，我们可以把这个参数作为第一次调用callback时的第一个参数，上面这个例子因为没有第二个参数，所以直接从数组的第二项开始，如果我们给了第二个参数为5，那么结果就是这样的：



```
[0, 1, 2, 3, 4].reduce(function(previousValue, currentValue, index, array){
 return previousValue + currentValue;
},5);
```

第一次调用的previousValue的值就用传入的第二个参数代替



14、**reduceRight（）**

reduceRight()方法的功能和reduce()功能是一样的，不同的是reduceRight()从数组的末尾向前将数组中的数组项做累加。

reduceRight()首次调用回调函数callbackfn时，prevValue 和 curValue 可以是两个值之一。如果调用 reduceRight() 时提供了 initialValue 参数，则 prevValue 等于 initialValue，curValue 等于数组中的最后一个值。如果没有提供 initialValue 参数，则 prevValue 等于数组最后一个值， curValue 等于数组中倒数第二个值。



```
var arr = [0,1,2,3,4];
 
arr.reduceRight(function (preValue,curValue,index,array) {
    return preValue + curValue;
}); // 10
```



15、**find()**

find()方法返回数组中符合测试函数条件的第一个元素。否则返回undefined 



```
var stu = [
    {
        name: '张三',
        gender: '男',
        age: 20
    },
    {
        name: '王小毛',
        gender: '男',
        age: 20
    },
    {
        name: '李四',
        gender: '男',
        age: 20
    }
]

stu.find((element) => (element.name == '李四'))
//返回结果为
//{name: "李四", gender: "男", age: 20}
```



16、**findIndex（）**

对于数组中的每个元素，findIndex 方法都会调用一次回调函数（采用升序索引顺序），直到有元素返回 true。只要有一个元素返回 true，findIndex 立即返回该返回 true 的元素的索引值。如果数组中没有任何元素返回 true，则 findIndex 返回 -1。

findIndex 不会改变数组对象。



```
[1,2,3].findIndex(function(x) { x == 2; });
// Returns an index value of 1.

[1,2,3].findIndex(x => x == 4);
// Returns an index value of -1.
```