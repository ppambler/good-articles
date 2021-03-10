# 超级详细的 js数组方法详解

原文：[超级详细的 js数组方法详解](https://juejin.cn/post/6907109642917117965#comment)

> 数组是 js 中最常用到的数据集合，其内置的方法有很多，熟练掌握这些方法，可以有效的提高我们的工作效率，同时对我们的代码质量也是有很大影响。

## 一、创建数组

### 1. 使用数组字面量表示法

``` js
var arr1 = []; //创建一个空数组
var arr2 = [20]; // 创建一个包含1项数据为20的数组
var arr3 = ["lily", "lucy", "Tom"]; // 创建一个包含3个字符串的数组
console.log(arr1, arr2, arr3) // [] [ 20 ] [ 'lily', 'lucy', 'Tom' ]
```

### 2. 使用 Array 构造函数

#### 无参构造

``` js
var arr4 = new Array(); // 创建一个空数组
console.log(arr4) // []
```

#### 带参构造

如果只传一个数值参数，则表示创建一个初始长度为指定数值的空数组

``` js
var arr5 = new Array(20); // 创建一个包含20项的数组
console.log(arr5) // [ <20 empty items> ]
```

如果传入一个非数值的参数或者参数个数大于 `1` ，则表示创建一个包含指定元素的数组

``` js
var arr6 = new Array("lily", "lucy", "Tom"); // 创建一个包含3个字符串的数组
var arr7 = new Array('23');
console.log(arr6, arr7) // [ 'lily', 'lucy', 'Tom' ] [ '23' ]
```

### 3. Array.of 方法创建数组(es6 新增)

ES6 为数组新增创建方法的目的之一，是帮助开发者在使用 Array 构造器时避开 js 语言的一个怪异点。

**Array.of()方法总会创建一个包含所有传入参数的数组，而不管参数的数量与类型。**

``` js
let arr8 = Array.of(1, 2);
console.log(arr8.length); // 2
let arr9 = Array.of(3);
console.log(arr9.length, arr9[0]); // 1、3
let arr10 = Array.of('2');
console.log(arr10.length, arr10[0]); // 1、'2'
```

> 给参数就是在为数组给元素……这并不是一种所谓的重载行为，不管给什么类型的参数，还是给多少个参数， `Array.of` 函数的行为都是一致的。

### 4. Array.from 方法创建数组(es6 新增)

在 js 中将**非数组对象转换为真正的数组**是非常麻烦的。在 ES6 中，将可迭代对象或者类数组对象作为第一个参数传入， `Array.from()` 就能返回一个数组。

``` js
function arga(...args) { //...args剩余参数数组,由传递给函数的实际参数提供
  console.log(args) //这是一个数组对象哈！可不是伪数组
  console.log(args.constructor) // Array 是 args 的构造函数
  console.log(args.__proto__) // 伪数组
  let arg = Array.from(args);
  console.log(arg);
}
arga('arr1', 26, 'from'); // ['arr1',26,'from']
```

> 这个例子，有点多此一举的意思……把一个数组转化成一个数组？？？

#### 映射转换

如果你想实行进一步的数组转换，你可以向 `Array.from()` 方法传递一个映射用的函数作为第二个参数。此函数会将数组对象的每一个值转换为目标形式，并将其存储在目标数组的对应位置上。

``` js
function arga1(a, b, c) {
  return Array.from(arguments, value => value + 1);
}
let arr11 = arga1('arr', 18, 'pop');
console.log(arr11); // ['arr1',18,'pop1']
```

> 对每个实参添加点东西

如果映射函数需要在对象上工作，你可以手动传递第三个参数给 `Array.from()` 方法，从而指定映射函数内部的 `this` 值

``` js
const helper = {
  diff: 1,
  add(value) {
    return value + this.diff;
  }
}

function translate() {
  //arguments 是一个对应于传递给函数的参数的类数组对象
  return Array.from(arguments, helper.add, helper);
}
let arr = translate('frank', 18, 'man');
console.log(arr); // ["frank1", 27, "man1"]
```

> 如果不指定 `this` ，那么 `add` 函数里边的 `this` 就是 `window` 了，毕竟我们是把一个函数引用值交给了 `Array.from` 函数内部去执行了……

## 二、数组方法

### 数组原型方法主要有以下这些

* `join()` ：用指定的分隔符将数组每一项拼接为字符串

* `push()` ：向数组的末尾添加新元素

* `pop()` ：删除数组的最后一项

* `shift()` ：删除数组的第一项

* `unshift()` ：向数组首位添加新元素

* `slice()` ：按照条件查找出其中的部分元素

* `splice()` ：对数组进行增删改

* `fill()` : 方法能使用特定值填充数组中的一个或多个元素

* `filter()` :“过滤”功能

* `concat()` ：用于连接两个或多个数组

* `indexOf()` ：检测当前值在数组中第一次出现的位置索引

* `lastIndexOf()` ：检测当前值在数组中最后一次出现的位置索引

* `every()` ：判断数组中每一项都是否满足条件

* `some()` ：判断数组中是否存在满足条件的项

* `includes()` ：判断一个数组是否包含一个指定的值

* `sort()` ：对数组的元素进行排序

* `reverse()` ：对数组进行倒序

* `forEach()` ：ES5 及以下循环遍历数组每一项

* `map()` ：ES6 循环遍历数组每一项

* `copyWithin()` : 用于从数组的指定位置拷贝元素到数组的另一个指定位置中

* `find()` : 返回匹配的值

* `findIndex()` : 返回匹配位置的索引

* `toLocaleString()、toString()` : 将数组转换为字符串

* `flat()、flatMap()` ：扁平化数组

* `entries() 、keys() 、values()` : 遍历数组

### 各个方法的基本功能详解

#### 1.join()

`join()`方法用于把数组中的所有元素转换一个字符串。

元素是通过指定的分隔符进行分隔的。默认使用逗号作为分隔符

```js
var arr = [1,'2',{}];
console.log(arr.join());   // '1,2,[object Object]'
console.log(arr.join("-"));   // '1-2-[object Object]'
console.log(arr);   // [ 1, '2', {} ]（原数组不变）
console.log(({}).toString())// '[object Object]'
```

通过**join()方法可以实现重复字符串**，只需传入字符串以及重复的次数，就能返回重复后的字符串，函数如下：

```js
function repeatString(str, n) {
// 一个长度为n+1的空数组用string去拼接成字符串,就成了n个string的重复 
// -> 间隔，3个苹果之间放了两个梨
	return new Array(n+1).join(str);
}
console.log(repeatString("abc", 3));   // 'abcabcabc'
console.log(repeatString("Hi", 5));   // 'HiHiHiHiHi'
```

> 字符串化一个个数组元素 -> 根据参数来拼接，不填参数，默认是把逗号作为梨子

#### 2.push()和 pop()

`push()` 方法**从数组末尾向数组添加元素**，可以添加一个或多个元素。

`pop()` 方法用于**删除数组的最后一个元素**并返回删除的元素。

```js
var arr = ["Lily","lucy","Tom"];
var count = arr.push("Jack","Sean");
console.log(count);  // 5
console.log(arr);   // ["Lily", "lucy", "Tom", "Jack", "Sean"]
var item = arr.pop();
console.log(item);   // 'Sean'
console.log(arr);   // ["Lily", "lucy", "Tom", "Jack"]
```

> `push`的返回值是原数组`push`之后的长度，而`pop`的返回值则是被弹出的那个元素

注意，我们对伪数组也可以使用这两个方法：

```js
function likeArray(a, b, c) {
  console.log(arguments); // {'0':1,'1':2,'2':3,length:3}
  console.log(Array.prototype.pop.call(arguments)); // 3
  console.log(arguments); // {'0':1,'1':2,length:2}
  console.log([].push.call(arguments, 4, 5)); // 4
  console.log(arguments); // {'0':1,'1':2,'2':4,'3':5, length :4}
}

likeArray(1, 2, 3);
```

#### 3.shift() 和 unshift()

`shift()` 方法用于**把数组的第一个元素从其中删除**，并返回第一个元素的值。

`unshift()` 方法可**向数组的开头添加一个或更多元素**，并返回新的长度。

```js
var arr = ['Lily', 'lucy', 'Tom'];
var count = arr.unshift('Jack', 'Sean');
console.log(count); // 5
console.log(arr); //["Jack", "Sean", "Lily", "lucy", "Tom"]
var item = arr.shift();
console.log(item); // 'Jack'
console.log(arr); // ["Sean", "Lily", "lucy", "Tom"]
```

> `unshift`，不转移

同样，对伪数组我们也可以这样做：

```js
function likeArray(a, b, c) {
  console.log(arguments); // {'0':1,'1':2,'2':3,length:3}
  console.log(Array.prototype.shift.call(arguments)); // 1
  console.log(arguments); // {'0':2,'1':3,length:2}
  console.log([].unshift.call(arguments, 4, 5)); // 4
  console.log(arguments); // {'0':4,'1':5,'2':2,'3':3, length :4}
}

likeArray(1, 2, 3);
```

据说，在 ECMAScript 中，长得像数组的对象，都可以被数组的方法所操作。这不仅仅局限于`pop/push`、`shift/unshift`这两组方法，其它很多方法都可以适用于这套规则。

> “当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。” -> 这个数据像鸭子，那么就可以用专属于鸭子的方法了！

#### 4.sort()

`sort()` 方法用于对数组的元素进行排序。

排序顺序可以是字母或数字，并按升序或降序。

默认排序顺序为按字母升序。



