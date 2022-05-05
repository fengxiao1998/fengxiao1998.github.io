---
title: async与await
date: 2020-06-30 15:03:18
tags:
---
## 一、什么是async、await

async函数是es7引入的，它是 Generator函数的语法糖（Generator函数是async函数的实现原理，这一次分享关注使用方法），它是异步操作的又一种实现。 

之前分享过Promise（[点击查看promise分享](https://alidocs.dingtalk.com/i/team/O5pXB6evwQ4aX7Zv/docs/O5pXBZob0bd4aX7Z)），async相当于是对Promise链式异步回调的优化版本。

总的来看异步操作的进化历程主要是：cb函数-> Promise的链式回调 -> async+Promise

async字面上理解一下就是异步的意思，await就是async wait，是异步等待的意思，wait的是resolve(data)的消息，并把数据data返回。

async相当于用同步的写法来写异步方法，不使用.then的方式链式回调

## 二、使用方法

#### 1. async

async是写在函数前面，用来声明这个函数里面包含异步操作。它返回一个promise对象

```
async function test () {
  console.log('我是函数内输出')
	return new Promise((resolve, reject) => {
  	resolve('输出我')
  })
}
test().then((data) => {
	console.log(data)
})
// 我是函数内输出
// 输出我
```

当返回值return的不是一个promise对象是，他会把它转化成promise

```
async function test () {
  console.log('我是函数内输出')
	return '输出我'
}
test().then((data) => {
	console.log(data)
})
// 我是函数内输出
// 输出我

// 上面的async相当于
async function test () {
	return Promise.resolve('输出我')
}
```

当这个函数没有return的时候，他会返回一个undefined

```
async function test () {
  console.log('我是函数内输出')
}
test().then((data) => {
	console.log(data)
})
// 我是函数内输出
// undefined
```

从上面看来，加了async声明的函数，和普通函数的区别就是返回值会被包装成promise对象，可以在then里面取到返回值

#### 2. await

await需要配合async使用，**只能写在async函数中, 用在普通函数中会报错。**

await 放置在Promise调用之前，await 强制后面的代码等待，直到Promise对象resolve，不用加then就能获取到resolve的结果，得到resolve的值作为await表达式的运算结果

```
async function test () {
	let output = await new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve('输出我')
    }, 1000)
  	})
  console.log(output)
}
test()
// Promise {<pending>} 这里没有接收test的输出，会打印在控制台这个promise输出（后面不列出来了）
// 1s后输出 -> 
// 输出我
```



从这里可以看到"我是函数内输出2",也是1s后和"输出我"一起输出的，说明函数执行到await这里的时候，它是阻塞的，必须等后面的异步有响应结果的时候，才会往后面执行



关于执行顺序，看下面

```
async function test () {
  console.log(1)
	let output = await new Promise((resolve, reject) => {
    console.log(2)
    setTimeout(() => {
      resolve(3)
    }, 1000)
  })
  console.log(output)
  console.log(4)
}
test().then(() => {
	console.log(5)
})
console.log(6)
// 输出顺序
```

这里可以看到await会阻塞，但是async函数并没有阻塞



```
1
2
6
3
4
5
```



#### 3. async和await结合

async函数里我们的异步方法，在按照代码顺序同步执行着

```
function output1 (num) {
	return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num+1)
    }, 1000)
  })
}
 function output2 (num) {
	return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num+2)
    }, 2000)
  })
}
async function test () {
  let num = 0
  console.log(num)
	let num1 = await output1(num)
  console.log(num1)
  let num2 = await output2(num1)
  console.log(num2)
}
test()
// 0 
// 1s后输出-> 1 (0+1)
// 2s后输出 -> 3 (1+2)
```



当后面await后面的跟的不是promise的时候，就不等待, 相当于同步

```
async function test () {
  console.log('我是函数内输出')
	let output = await '输出我' // 就直接相当于 let output = '输出我'
  console.log(output)
}
test()
// 我是函数内输出
// 直接输出 -> 输出我
```

#### 4. 错误处理

当async函数中出现错误时，会中断后面的执行，无论是Promise reject的数据还是逻辑报错，都会中断。所以需要对错误做处理

async的函数里面报错，相当于返回的promise被reject了，所以错误能通过函数的catch捕获到

```
async function test () {
  console.log('我是函数内输出')
	let output = await new Promise((resolve, reject) => {
      throw new Error('出错了') // 模拟报错
      resolve('输出我')
  })
  console.log(output)
  return '返回了'
}
test().then((data)=>{
  console.log(data)
}).catch((err)=> {
	console.log(err, '进入catch了')
})
// 我是函数内输出
// Error: 出错了    '进入catch了'
```



也可以使用try catch 处理错误

```
async function test () {
  try {
    console.log('我是函数内输出')
    let output = await new Promise((resolve, reject) => {
        throw new Error('出错了') // 模拟报错
        resolve('输出我')
    })
    console.log(output)
  } catch (err) {
  	console.log(err, '进入catch了')
  }
}
test()
// 我是函数内输出
// Uncaught Error: 出错了  '进入catch了'
```

从上面可以看到，出错后都不会走后面的代码。



如果想要继续往下执行，对await异步里的错误做完处理就能继续执行后面的代码

```
async function test () {
  console.log('我是函数内输出')
	let output = await new Promise((resolve, reject) => {
      throw new Error('出错了') // 模拟报错
    	reject(234)
  }).catch(err => {console.log(err, 'await的报错catch')})
  console.log('报错后面的输出')
}
test()
// 我是函数内输出
// VM9420:6 Error: 出错了
    at <anonymous>:4:13
    at new Promise (<anonymous>)
    at test (<anonymous>:3:21)
    at <anonymous>:9:1 "await的报错catch"
// 报错后面的输出
```



## 三、注意事项汇总

#### 1. await只能在async函数内部使用，不能放在普通函数里面，否则会报错。

#### 2. await关键字后面跟Promise对象，会等待后面的resolve（data）之后，才会向下执行，这里是阻塞的。

#### 3. await命令后面的Promise对象，运行结果可能是rejected，所以要对它的错误做处理。

#### 4. 多个await命令后面的异步操作，如果不存在继发关系，最好让它们同时触发，否则是串联的等待。



## 四、优点

#### 1. 方便级联调用，同步代码和异步代码可以一起写，看着逻辑清晰

同步代码该怎么写继续怎么写，异步过程需要包装成一个Promise对象放在await关键字后面，读下来就像是同步的

```
function takeLongTime(n) {
    return new Promise(resolve => {
        setTimeout(() => resolve(n + 200), n);
    });
}

function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(n) {
    console.log(`step2 with ${n}`);
    return takeLongTime(n);
}

function step3(n) {
    console.log(`step3 with ${n}`);
    return takeLongTime(n);
}
```



```
// promise链式写法
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => step2(time2))
        .then(time3 => step3(time3))
        .then(result => {
            console.log(`result is ${result}`);
        });
}
doIt();

// async函数写法
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time2);
    const result = await step3(time3);
    console.log(`result is ${result}`);
}
doIt();

// step1 with 300
// step2 with 500
// step3 with 700
// result is 900
```

下面的例子更加明显，需求改成每一个步骤都需要之前每个步骤的结果

```
function step1(n) {
    console.log(`step1 with ${n}`);
    return takeLongTime(n);
}

function step2(m, n) {
    console.log(`step2 with ${m} and ${n}`);
    return takeLongTime(m + n);
}

function step3(k, m, n) {
    console.log(`step3 with ${k}, ${m} and ${n}`);
    return takeLongTime(k + m + n);
}
// Promise方式调用
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then(time2 => {
            return step2(time1, time2)
                .then(time3 => [time1, time2, time3]);
        })
        .then(times => {
            const [time1, time2, time3] = times;
            return step3(time1, time2, time3);
        })
        .then(result => {
            console.log(`result is ${result}`);
            console.timeEnd("doIt");
        });
}

// async/await方式调用
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    const time3 = await step2(time1, time2);
    const result = await step3(time1, time2, time3);
    console.log(`result is ${result}`);
    console.timeEnd("doIt");
}
// step1 with 300
// step2 with 800 = 300 + 500
// step3 with 1800 = 300 + 500 + 1000
// result is 2000
// doIt: 2907.387ms
```



#### 2. 更方便的参数传递

Promise的then函数只能传递一个参数，虽然可以通过包装成对象来传递多个参数，但是会导致传递冗余信息，频繁的解析又重新组合参数，比较麻烦；async/await没有这个限制，可以当做普通的局部变量来处理，用let或者const定义的块级变量想怎么用就怎么用，想定义几个就定义几个，完全没有限制，也没有冗余工作

```
// promise链式写法
function doIt() {
    console.time("doIt");
    const time1 = 300;
    step1(time1)
        .then((time2) =>{
        	let output = 1
        	return step2(time2)
    	})
        .then((time3) => {
        	output += 1
        	console.log(output)
        	return step3(time3)
    	})
        .then(result => {
            console.log(`result is ${result}`);
        });
}
doIt();

// async函数写法
async function doIt() {
    console.time("doIt");
    const time1 = 300;
    const time2 = await step1(time1);
    let output = 1
    const time3 = await step2(time2);
    output += 1
    console.log(output)
    const result = await step3(time3);
    console.log(`result is ${result}`);
}
doIt();
```

## 五、常见场景

#### 1.两个接口请求完，执行某操作

```
// 两个接口相关联，请求完1才能请求2

async function exc1 () {
    console.log('exc1 start:',Date.now())
    let res1 = await fetch1(url);
    let res2 = await fetch2(res1); // 依赖1的接口返回新的url
    console.log('exc1 end:', Date.now())
}

// 并发请求， 两个接口不关联，两个请求完的时候之后后面的逻辑
async function exc2 () {
    console.log('exc2 start:',Date.now())
    let [res1, res2] = await Promise.all([fetch1(url), fetch2(url)])
    console.log('exc2 end:', Date.now())
}

exc1();

exc2();
```



#### 2.根据是否超时，做处理

```
function timeOut (delay) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
        	resolve(false)
        }, delay)
    })
}
async function exc1 () {
	let flat = await Promise.race([fetch(url), timeOut(delay)])
    // 根据是否超过规定的时间，做不通的处理
}
```



#### 3. 一个接口请求失败的话，再尝试请求一定次数

```
function getImg (url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(false)
    }, 3000)
  })
}
async function fn3() {
  const times = 3
  let url = 'xxx'
  for (let i = 0; i < times; i++) {
    let img = await getImg(url).catch(() => {})
    console.log(img)
    if (img) {
      break;
    }
  }
}
fn3()
```



其它场景欢迎补充....



关于Generator函数，大家可以去看下阮一峰的es6