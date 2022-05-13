---
title: JavaScript性能优化之Set()
date: 2022-05-12 15:03:18
tags: [前端性能优化,JavaScript性能优化,Set]
---

## 前言：

前端性能优化一直也是老生常谈的内容了，当初面试时，笔者被问起【性能优化时】，笔者往往也是诸如:"动态加载"，”懒加载“，”大型图片使用图片压缩预展示“等等，这些往往都是宏观上的思路，对于细节问题，笔者似乎没有过多的重视过，现在则是时候研究了。

## 起因：

在LeetCode刷题时，笔者遇到如下问题：

![image-20220512105404621](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220512105404621.png)

笔者一开始的想法如下：

```javascript
var containsDuplicate = function (nums) {
    const newArr = []
    for (i = 0; i < nums.length; i++) {
        if (newArr.includes(nums[i])) {
            return true
        }
        newArr.push(nums[i])
    }
    return false
};
```

很简单的思路，笔者在此不多做叙述，重点不在这里，结果是正确的。反馈如下：

![image-20220512105606765](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220512105606765.png)

看到这里，笔者愣了一下，执行用时只击败了这么点人？出于学习的心里，笔者来到了官方解题思路下面，看到官方的代码如下：

```javascript
var containsDuplicate = function (nums) {
    const set = new Set();
    for (const x of nums) {
        if (set.has(x)) {
            return true;
        }
        set.add(x);
    }
    return false;
};
```

笔者一开始不屑的笑了笑，思路和我一样，无非是他用了逼格更高的`Set()`，不妨看看他的代码运行结果。

![image-20220512105856892](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/image-20220512105856892.png)

啊？

![下载](https://heyyyfx.oss-cn-hangzhou.aliyuncs.com/img/blog/%E4%B8%8B%E8%BD%BD.jpg)

这咋差了这么多时间？出于好奇，笔者去查阅了相关资料，在廖雪峰老师的博客下看到了这篇文章。

文章地址：https://www.liaoxuefeng.com/wiki/1022910821149312/1023024181109440

这篇文章是将Map与Set放在一起讲解，文中提及，`Map`与`Set`都是【键值对结构】而`Set`是一组key的集合，但不存储value。由于key不能重复，所以，在`Set`中，没有重复的key。这也是Set用来去重的另一个作用。

通过这些信息我们可以知道，与数组不同，数组是索引值的，Set是一种类似对象的结构，当你要查询数组中是否有某个值时，需要将整个数组走一遍，时间复杂度为O(n)，n为数组长度。但如果是键值对的形式，时间复杂度为O(1),这就是它省时的秘密。

**附：** Set去重时要注意对象无法去重，函数无法去重。

