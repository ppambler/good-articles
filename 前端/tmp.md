1）浅合并是什么？

合并的意思：

1. v：把几个事物合成一个事物 -> 精简机构，合并科室
2. v：由一种疾病引发另一种疾病；（多种病）同时发作。 -> 合并症

简单来说，把 5 班的同学合并到 3 班去，这样 5 班就不存在了！

对于两个对象而言，所谓浅合并就是把第一层的键和值进行合并和替换：

1. 合并指的是 -> obj2 有 obj1 没有的属性，那就给 obj1
2. 替换指的是 -> obj2、obj1 都有的属性，那 obj2 的属性值就会替换掉 obj1 的

``` js
obj1 === Object.assign(obj1, obj2) // true
```

> 如果 obj1 不是对象，如 5、'6'这样，那么这些基本类型值会被包装成对象，再进行合并，如 `'6'` 会包装成字符串对象，即它的构造器是 `String`

同理，深合并就是，第二层、第三层等也会进行合并和替换

➹：[web 前端高级 JavaScript - 对象的深合并与浅合并_lixiaosenlin 的专栏-CSDN 博客](https://blog.csdn.net/lixiaosenlin/article/details/109785219)

2）什么叫不可变对象？

不可变的原理很简单，就是**不修改原有对象**，而是通过产生新的来代替原来的，作用也非常单一，就是零副作用。听起来有点别扭，举个例子，你在一个循环里面遍历一个数组，遍历过程中还修改了原数组的数据，这就会带来一些副作用，比如死循环之类的。

immer 和 immutable.js 都只是基于此做了一些封装，让“不可变”写起来更爽而已。

> 简单来说，不要修改对象里边的属性

➹：[immer.js不可变对象的用途是什么呢？什么情况需要使用呢？ - SegmentFault 思否](https://segmentfault.com/q/1010000022867342)

➹：[JavaScript浅析 -- 可变对象和不可变对象 - 简书](https://www.jianshu.com/p/327de4c87991)

➹：[不可变数据结构（immutable data） · Issue #33 · sunyongjian/blog](https://github.com/sunyongjian/blog/issues/33)

➹：[从JS对象开始，谈一谈前端“不可变数据”和函数式编程](https://juejin.cn/post/6844903470718255118)

3）对象里边出现等号是什么神仙语法？

``` js
{
  owner = {
    name: '老干部',
    age: 30
  }
}

// {name:'老干部',age:30}
```

最外层的这个`{}`是块级作用域哈！

文章里边是这样的：

``` js
this.state = {
  owner = {
    name: '老干部',
    age: 30
  }  
}
```

显然是`owner:{}`

➹：[Object initializer - JavaScript | MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)

4）`PureComponent`是什么？

