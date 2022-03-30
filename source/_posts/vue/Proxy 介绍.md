### Proxy 介绍

```
let p = new Proxy(target, handler);

参数：
	-1. target 用Proxy包装的目标对象（可以是任何类型的对象，包括原生数组，函数，甚至另一个代理）
  -2. handler 一个对象，其属性是当执行一个操作时定义代理的行为的函数。
  
注意：
	要想Proxy生效， 必须针对Proxy实例进行操作，而不是对原对象target
```



### Proxy Demo

下面是一个来自MDN的demo:

```
let validator = {
	set (obj, prop, value) {
  	if (prop === 'age') {
    	if (!Number.isInteger(value)) {
      	throw new TypeError('The age is not an interger');
      }
      if (value > 200) {
      	throw new RangeError('The age seems invalid');
      }
    }
    
    obj[prop] = value
  }
}

let preson = new Proxy({}, validator);
person.age = 100;
// 100
person.age = 'young'; 
// 抛出异常: Uncaught TypeError: The age is not an integer

person.age = 300; 
// 抛出异常: Uncaught RangeError: The age seems invalid
```



### Proxy 和 defineProperty

Proxy在vue3中主要是代替Object.defineProperty()实现了响应式, 下面先简单的对比一下他们的区别：

#### 1. 数据劫持

（通常使用Object.defineProperty劫持对象的访问器，在值变化时获取变化，并进行操作）

```
const data = {
	habit: ''
}
function play(habit) {
	if (habit === 'bs') {
  	console.log('中国男篮使我放弃了篮球的爱好！')
  } else {
  	console.log('我只爱中国女排！')
  }
}

Object.keys(data).forEach(function(key) {
	Object.defineProperty(data, key, {
  	configurable: true,
    enumerable: true,
    get () {},
    set (val) {
    	console.log(val)
      play(val)
    }
  })
})

// 当我们对data中的habit进行赋值的时候就会触发
data.habit = 'bs'
// bs
// 中国男篮使我放弃了篮球的爱好！
```



一个完整的数据劫持基本有以下几点：

```
一、利用Observer针对对象(属性)进行劫持，在属性发生变化后通知订阅者
二、Compile解析器根据Directive进行渲染
三、Watcher会连接Observer和Compile，使得数据变化促使视图变化
```



#### 2. 基于Object.defineProperty的双向绑定

```
简易版：
const ready = {};
Object.defineProperty(ready, 'start', {
	get () {
  	console.log('This is getter')
  },
  set (val) {
  	console.log('This is setter:' + val)
    // 这里可以用来操作DOM
  }
})

xx.addEventListener('keyup', function(e) {
	ready.start = e.target.value; // 双向绑定
})


升级版：
let uid = 0;
----------- 负责存储订阅者和消息的分发
class Dep {
	constructor() {
    // 设置id, 用于区分
  	this.id = uid ++;
    // 存储订阅事件
    this.subs = [];
  }
  depend() {
  	Dep.target.addDep(this)
  }
  // 添加订阅事件
  addSub(sub) {
    this.subs.push(sub);
  }
  notify() {
    // 通知所有的订阅事件(Watcher)，触发订阅者的相应逻辑处理
    this.subs.forEach(sub => sub.update());
  }
}

---------- 监听者Observer
class Observer {
	constructor(value) {
    this.value = value;
    this.walk(value);
  }
  // 遍历属性值并添加监听状态
  walk(value) {
    Object.keys(value).forEach(key => this.convert(key, value[key]));
  }
  convert(key, val) {
    defineReactive(this.value, key, val);
  }
}

function defineReactive (obj, key, val) {
  	const dep = new Dep();
   	// 给当前属性的值添加监听
   	let chlidOb = observe(val);
  	Object.defineProperty(obj, key, {
    	enumerable: true,
      configurable: true,
      get: () => {
        // 如果Dep类存在target属性，将其添加到dep实例的subs数组中
        // target指向一个Watcher实例，每个Watcher都是一个订阅者
        // Watcher实例在实例化过程中，会读取data中的某个属性，从而触发当前get方法
        if (Dep.target) {
          dep.depend();
        }
        return val;
      },
      set: newVal => {
        if (val === newVal) return;
        val = newVal;
        // 对新值进行监听
        chlidOb = observe(newVal);
        // 通知所有订阅者，数值被改变了
        dep.notify();
      }
    })
}
function observe(value) {
  // 当值不存在，或者不是复杂数据类型时，不再需要继续深入监听
  if (!value || typeof value !== 'object') {
    return;
  }
  return new Observer(value);
}

---------- 订阅Watcher
class Watcher {
	constructor(vm, expOrFn, cb) {
  	this.depIds = {};
    this.vm = vm;
    this.cb = cb;
    this.expOrFn = expOrFn;
    this.val = this.get()
  }
  update () {
  	this.run();
  }
  addDep(dep) {
  	if (!this.depIds.hasOwnProperty(dep.id)) {
      dep.addSub(this);
      this.depIds[dep.id] = dep;
    }
  }
  run() {
    const val = this.get();
    console.log(val);
    if (val !== this.val) {
      this.val = val;
      this.cb.call(this.vm, val);
    }
  }
  get() {
    // 当前订阅者(Watcher)读取被订阅数据的最新更新后的值时，通知订阅者管理员收集当前订阅者
    Dep.target = this;
    const val = this.vm._data[this.expOrFn];
    // 置空，用于下一个Watcher使用
    Dep.target = null;
    return val;
  }
}
```

#### 

#### 3. Object.defineProperty的不足

```
1. 无法监听数组
在Vue中只对以下八种方法做了优化push() pop() shift() unshift() splice() sort() reverse()
2. vm.items[indexOfItem] = newVal 无效
3. 只能劫持对象的属性，且需要层层遍历
```



#### 4. 基于Proxy的双向绑定

Proxy可以直接监听对象而不是属性

先看一个简版的demo:

```
const input = document.getElementById('input')
const p = document.getElementById('p')
const obj = {}

const newWay = new Proxy(obj, {
	get (target, key, rev) {
  	console.log(`U get ${key}!`)
    return Reflect.get(target, key, rev)
  },
  set (target, key, value, rev) {
    if (key === 'text') {
      input.value = value
      p.innerHTML = value
    }
    console.log(target, key, value, rev)
    return Reflect.set(target, key, value, rev)
  }
})
// 这里可以直接监听一个obj对象

input.addEventListener('keyup', function(e) {
 newObj.text = e.target.value
})
```



#### 5. Proxy的优点

Proxy可以直接监听数组的变化，且包含（apply、ownKeys、deleteProperty、has）等Object.defineProperty不具备的方法

```
const arr = [1, 2, 3, 4]

const newArr = new Proxy(arr, {
	get (target, key, rev) {
  	console.log(key)
    return Reflect.get(target, key, rev)
  },
  // 
  set (target, key, value, rev) {
  	if (key !== 'length') {
    	// 执行改变
    }
    return Reflect.set(target, key, value, rev)
  }
})

newArr.push(6)
```



这样一来，在vue3中，由于数据相应发生了变化，$set将不再会被频繁的使用到了。