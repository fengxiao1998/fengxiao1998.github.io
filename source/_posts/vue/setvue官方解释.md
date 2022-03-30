# 一，vue官方解释

[https://cn.vuejs.org/v2/guide/list.html#%E5%AF%B9%E8%B1%A1%E5%8F%98%E6%9B%B4%E6%A3%80%E6%B5%8B%E6%B3%A8%E6%84%8F%E4%BA%8B%E9%A1%B9](https://cn.vuejs.org/v2/guide/list.html#对象变更检测注意事项)

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/10c81c38-a4c5-452f-be70-88f475dbd084.png)

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/fba60172-87fa-4d55-bbdb-ec2b4760450e.png)

这里要仔细说的一点事，$set是向一个对象中添加一个新响应性属性所以，请确定这个属性一定要是新的属性，一旦这个对象上面被你提前通过data.xx 或者data[xx]赋值了一个属性,你再去set它，它将不会变成新的响应式属性。

（同时要注意的是$set一个新属性生效时，会触发这个对象上 引用属性的watch的更新 ）



# 二，怎么判断一个对象上的属性是否是响应式的



![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/fadeb573-f614-46b2-8764-ee2d700d6121.png)

可以看到data 中的cat 是有age这个提前声明的属性的

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/c53c48a5-ebcf-4567-a17a-cae9415ad858.png)

控制台上可以看到

vue 是重写了cat .age 的 get 和 set 的



下面我们通过.的方式增加一个属性

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/b45b6f4b-ebbe-4bce-876c-bacc6c129ff5.png)

可以看到赋值成功，可是vue却没有为name 重新定义get 和 set 



下面我们通过$set 添加一个属性

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/a6e42038-5f78-4f07-a9e4-1f4d5c38e3dc.png)

可以明显的看出是多了一个color的 get 和 set

为了验证前面的说法，我们下面set一下之前. 上去的name属性

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/fd78603b-28de-46ac-8590-305020ba5061.png)

可以看到set是能够改变name的值的，可是却还是没有将name变成响应式属性此处验证了我们上面的说噶。



# 三，源码篇

## 对象的$set

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/2c3e5144-6b17-4b65-9437-f1d37934f2ed.png)

这里很明显的可以看出，当一个属性已经在这个目标上时仅仅只会给这个属性赋值而已

## 数组的$set补充

这里对观测的target进行判断，如果是数组的话，则使用target.splice方法进行处理。

key值进行了判断取数组长度、传入的值

splice方法可以删除，填充，修改数组等等操作。最重要的是可以修改原数组。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/7jP2lR7kV2bmn8g5/img/0eb7837b-797e-4070-94d1-c3d4e009536f.png)