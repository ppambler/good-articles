# JavaScript核心知识点

## 1.栈

`栈`者,存储货物或供旅客住宿的地方,可引申为仓库

**栈**（类似好多的计算机系统中的概念）是计算机科学工作者抽象出来的概念（客观世界中只有内存这个东西），所谓科学，其实是从认识和实践客观事物中抽象出一定规律，并在这个规律中进行演绎，最后反过来指导认识和实践客观事物的过程。

> 所谓科学，即类似于取之于民，用之于民

### 1.1 数据结构中的栈

- 栈是一组数据的存放方式,特点是先进后出，后进先出

![instack](https://gitee.com/ppambler/blog-images/raw/master/fe/VZpCelM6cbwSEHX.png)

![outstack](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516203728.png)

> 想想羽毛球桶 -> 拿羽毛球的规则是怎样的……

| 方法名 | 操作                                 |
| :----- | :----------------------------------- |
| push() | 添加新元素到栈顶                     |
| pop()  | 移除栈顶的元素，同时返回被移除的元素 |

```ts
class Stack {
    private items: number[] = [];
    // 添加元素到栈顶，也就是栈的末尾
    push(element: number) {
        this.items.push(element);
    }
    // 栈的后进先出原则，从栈顶出栈
    pop(): number | undefined {
        return this.items.pop();
    }
}

let stack = new Stack();
// stack.push(1);
// stack.push(2);
// stack.push(3);
console.log(stack.pop());
// console.log(stack.items) // items 只能在 class 里边被访问
```

### 1.2 代码的运行方式 

- 表示函数的一层层调用

![image-20210516205937505](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516205937.png)

> 代码执行完，意味着 `anonymous` 也就消失了 -> 这 TM 就是 Call Stack

### 1.3 内存区域

- 栈也是存放数据的一种内存区域
- 程序运行的时候，需要内存空间存放数据。一般来说,系统会划分出两种不同的内存空间：一种叫做stack(栈)，另一种叫做heap(堆)
  - stack是有结构的，每个区块按照一定次序存放，可以明确知道每个区块的大小
  - heap是没有结构的，数据可以任意存放。因此，stack的寻址速度要快于heap
- 只要是局部的、占用空间确定的数据，一般都存放在stack里面，否则就放在heap里面,所有的对象都存放在heap

> **内存对于cpu来说就是一段连续的数组（整个内存容量）** -> 栈就在这个连续的数组中的某一段

```js
function task() {
    var a = 1;
    var b = 2;
    var c = {
        name: 'zhufeng',
        age: 10
    }
}
task();
```

![memoryposition](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516210436.jpeg)

> 如果一行就是一个数组单元，那么整个仓库就是个内存条 -> 如果一行就是一个堆 -> 如果一行就是一个对象，像 `A1` 就是它的内存地址，那么整个仓库就是一个堆

## 2. 队列

- 队列是一种操作受限制的线性表
- 特殊之处在于它只允许在表的前端进行删除操作，而在表的后端进行插入操作
- 进行插入操作的端称为队尾，进行删除操作的端称为队头
- 因为队列只允许在一端插入,在另一端删除，所以只有最早进入队列的元素才能最先从队列中删除,故队列又称为先进先出线性表

![inqueue.png](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516220129.png)

![outqueue.png](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516220143.png)

```tsx
class Queue {
    private items: number[] = [];
    // 添加元素到栈顶，也就是栈的末尾
    enqueue(element: number) {
        this.items.push(element);
    }
    dequeue() {
        return this.items.shift();
    }
}

let queue = new Queue();
queue.enqueue(1);
queue.enqueue(2);
queue.enqueue(3);
console.log(queue.dequeue());//1
```

## 3. 数据类型

JS中有七种基本数据类型

- 六种基本数据类型 Boolean Number String Null Undefined Symbol
- 一种引用类型Object -> `{} [] /^$/ new Date() Math`

| 类型      | 值          |
| :-------- | :---------- |
| Boolean   | true或false |
| Null      | null        |
| Undefined | undefined   |
| Number    | 数字        |
| String    | 字符串      |
| Symbol    | 符号类型    |

## 4. 执行上下文

### 4.1 如何存储

- 当函数运行时，会创建一个执行环境，这个执行环境就叫执行上下文(Execution Context)
- 执行上下文中会创建一个对象叫作变量对象(Value Object),基础数据类型都保存在变量对象中
- 引用数据类型的值保存在堆里，我们通过操作对象的引用地址来操作对象

![image-20210516234530226](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516234530.png)

![valueobject](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517000413.png)

> ECStack -> EC(Global) -> EC(Local)
>
> 代码在执行前，浏览器会创建一个可执行环境，即 `ECStack`（执行环境栈，分配一块栈内存给程序使用） -> `script`里边的代码执行，在执行前会初始化这件事情：`GO`、`VO`、`[[scope]] -> VO` ，执行开始会确定 `scopeChain -> 上一级的空 VO`
>
> 函数的创建，会初始化`[[scope]]`（确定自己是哪个作用域内被创建的） -> 而函数的执行，会确定 `scopeChain -> 上一级的 VO `、`VO` 、`this`
>
> 闭包 -> 保护作用（外界不能访问函数内部的 `VO`）、保存作用（函数执行完，EC 会被释放，如果 `VO`里边有东西 `xxx` 被全局某个东西间接使用着，那么这个 `xxx` 是会被保存下来的）

### 4.2 如何复制

