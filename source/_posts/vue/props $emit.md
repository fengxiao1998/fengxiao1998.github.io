# 一. props $emit

**向下:** 在vue中可以通过 v-bind 将父组件数据传递给子组件（前提是子组件需要 什么props 接口，未生命的key 会存放在子组件api $attrs 中）

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/P4maOgKzG5NmOWNX/img/806b690f-943f-495d-a835-f51ab1a4d263.png)

**向上:** 子组件向上最简单的是就是this.$emit('eventName',arg),此方法可以理解为触发自身的“eventName” 事件，既然是事件肯定需要订阅者 （在父组件中改组件的template上 eventName=“handler” 即为订阅该组件上的此事件）

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/P4maOgKzG5NmOWNX/img/16704445-2073-4397-9ec3-5f733aead002.png)



**高级:** 在此基础上我们扩展一下，再我们的子组件是表单类组件（用户填写选择数据的输入选择组件）时，我们需要大量的数据通信，父组件管理表单数据，子组件回显表单数据，数据改变时通知父组件，这无疑是一种比较普遍的场景，此时我们劲量推荐使用 v-model 双向绑定来实现( 即父组件控制子组件的展示，子组件在值改变时，立刻同步修改父组件中的数据 ) 实际上v-model 仅仅只是上述传值的一种语法糖(简写) v-model 默认在传递一个 key为 value的prop 给子组件，子主键中默认触发 $emit('input',arg) 父组件中不需要额外处理该input 事件即可实现将arg 同步赋值给value（前提注意子组件一定要额外声明 该value props）。



这个时候可能你会想说，有时候你不想使用value 这个props，比较values的语义性不适应于很多场景，比如复选框一般采用checked。那么没关系，我们只需要在上面的基础上 给该子组件添加一个 model属性



```
 Vue.component('my-checkbox', {
  model: {
    prop: 'checked',
    event: 'change'
  },
  props: {
    // this allows using the `value` prop for a different purpose
    value: String,
    // use `checked` as the prop which take the place of `value`
    checked: {
      type: Number,
      default: 0
    }
  },
  // ...
})
```

可以看到model 中有两个固定要实现的key 一个是prop 表示你想要双向控制的props 名，event 表示触发双向绑定的事件名（这里额外说一下：加了model 也是需要显示的什么该 props 如上面代码的 第10行）此时也能实现双向绑定的效果而且更加灵活。



# 二，.sync 更加简易的双向绑定。

上面的例子高级中我们介绍了props 传值，$emit 响应事件向父传值，也讲了高级版双向绑定通信，可是这里我们需要思考一下，v-mode 这么好用难道没有弊端了吗？ 其实v-mode的弊端还是有的，就是v-model 在自定义时需要额外的声明model 这个属性来定义双向绑定特性，二是一个组件上只能拥有一个v-model 指令。当出现我们需要快捷的双向绑定两个属性的时候怎么去处理呢。

比如我们有一个弹窗输入主键，无疑改组件我们需要实现两个功能：

1，show 去控制弹窗显影。

2，value 去双向绑定输入的值。

这里我们思考一下，我们的子组件的show是不是也是一种类双向绑定，父组件通知子组件是否显影，子组件关闭时告诉父组件自己关闭了。此时我们无疑需要两个双向通信来简化代码。

来看看.sync的官网介绍

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/P4maOgKzG5NmOWNX/img/7d1a699b-de7b-4d16-950b-692d1e3c75ea.png)

子组件中的向上双向是通过$emit('update: propsName',arg) 相比起来是不是更加简单，而且更加灵活。但是这里大家注意一下，**v-model 和 .sync 的绑定值不能是表达式哦 。**



# **三，vuex超远通信。**

vuex是一种全局的单例数据状态（其实就是数据）关联工具。有一些全局性的数据各处都有可能调用的。（比如当前用户的用户名什么的。）首先首页的AppBar 我们需要显示该用户的用户名。而其他地方也有可能用到，这个时候有可能两个组件相隔很远，比如一个修改用户名的弹窗，修改完之后如果还需要找到这个AppBar 去给他传值，的话，props+$emits 的弊端就出现了，他们只能通过 子父关系 一级一级一级一级的传啊，传啊，更有可能他们是兄弟关系，还得通过共同祖先来中专这个值，极其复杂切不易维护，这个时候我们只需要将这样的数据交给 vuex 去全局统一处理（其实可以把vuex 理解为全局变量 不过它是响应式的）

弊端: 当然vuex 也有弊端，刚刚说到了vuex 是类似 window 的，那么作为引用数据，只要该数据发生变化，所有引用改数据的组件都会发生变化，再你的组件在页面上多次使用但是又想切断引用的时候，它就会导致页面絮乱，极其难以管理。总结一下就是， vuex 劲量用在 页面上组件 1 v 1的 组件之间通信，比如 appBar 只有一个。

这里就不代码举例了。



# 四，通过实例通信。

我们知道在子组件中可以通过$parent 直接获取到父亲实例，在父组件中也可以通过$refs['child'] 来获取到子组件的实例，在设计form组件时，都是父组件去主动调用 子组建的 validate 方法去校验子组件同时拿到改form 的formData。这种方法也不失为一种好方法，一时不用刻意去props $emit 去固定通信。在数据传递不是很频繁的时候可以考虑。

1.this.$parent.$emit('xxx', arg) + this.$parent.$on('xxx', (arg) => { ... }) --兄弟通信



2.this.$root.$emit('xxx', arg) + this.$parent.$on('xxx', (arg) => { ... }) --万能



3.this.$[provider].$emit('xxx', arg) + this.$[inject].$on('xxx', (arg) => { ... }) --递归场景 获取其他任何场景都可以





# 五，空实例传值（又名bus传值）

上面已经介绍了很多种传值，再介绍一种相关很远，又没多大亲戚关系的传值。

第一种传值中介绍过组件的事件事件是 A.$emit(’eventName‘) X.$on('eventName') 即可实现事件订阅。

那我们是不是可以通过一个空实例去作为这个中间件 去交接处理我们的 组件通信呢？ 答案肯定是可以的，可以想象成它（空实例）就是一个接线员，在你的事件触发时帮你连接 对应的订阅者。

比如下面有两个组件A B 一个空实例 bus 

在A组件的created 中 添加订阅



```
{
  created(){
    bus.$on('事件A',()=>{
      // A事件的处理
    })
  }
}
// 组件A
```



在组件B 中某个时机去触发改bus 事件

```
{
  methods:{
    // 某click事件
    onclick(){
      bus.$emit('事件A',"我被点击了")

    }
  }
}
// 组件B
```



这样就可以实现没有关系的组件通信。

**!!! 注意:** 需要注意的是，这个组件虽然灵活，但是一定要注意，在组件销毁时一定要用off销毁该事件，这里的销毁也要额外注意。



```
// bad
{
  created(){
    bus.$on('事件A',()=>{
      // A事件的处理
    })
  },
  beforeDestroy() {
     bus.$off('事件A')
  },
}

// good
{
  methods:{
    // A事件的处理
  	handler(){
			
		}
  },
  created(){
    bus.$on('事件A',handler)
  },
  beforeDestroy() {
     bus.$off('事件A',handler)
  },
}
```

上面的代码为什么要这样写呢？

1，有可能项目中多个组件订阅了bus 上的事件A 如果直接$off 是会关掉所有事件A的订阅，无疑是我们不想的。这样会导致其他组件对 bus上事件A 的订阅失效。

2, $off 关闭指定方法只会关掉自己的组件上bus 事件A 的订阅。这样才是对的。



扩展：这里需要思考一定bus 一定要是一个 new Vue 实例吗？

这个回答你们自己去想哦