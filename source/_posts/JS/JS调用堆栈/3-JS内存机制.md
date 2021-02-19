---
title: 3.JS内存机制
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/js.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/70.jpg'
date: 2021-01-05 19:10:30
categories:
    - JS
    - JS调用堆栈
tags:
    - JS内存回收
    - 内存泄漏
---



&emsp;&emsp;JS 内存空间分为 **栈**、**堆**、**池（一般归类为栈中）**。其中**栈**存放变量，**堆**存放复杂对象，**池存放常量**，因此也叫常量池。



# 变量的存放

&emsp;&emsp;JS 使用了传统的**堆栈**来保存变量：

- **基本类型**：保存在**栈**内存中，这些类型在内存中有**固定的大小**，通过按值来访问。基本数据类型一共有 6 种：`Undefined`、`Null`、`Boolean`、`Number`、`String`、`Symbol`（ES6）
- **引用类型**：保存在**堆**内存中，因为其**大小不固定**，因此不能将它们保存在栈中，但其保存的位置的地址大小是固定，因此其**访问地址保存在栈中**。当查询引用类型时，先从栈中读取内存地址，然后再通过这个内存地址找到堆中的值。这种访问方式我们称为**按引用访问**。

<img src="https://gitee.com/ylea/imagehost/raw/master/img/2019-07-24-060214.png" alt="img" style="zoom:80%;" />

&emsp;&emsp;在计算机中，栈的运算效率比堆高，而 `Object` 是一种复杂的结构且可以拓展：数组可扩充、对象属性可添加，可增删改。为了不影响栈的效率，因此将它放到堆中并以引用的方式查找到堆中的实际对象再进行操作。

&emsp;&emsp;所以查找引用类型值的时候回先去栈中查找再去堆中查找。



==鼠标选中查看答案==

**问题1**

```js
var a = { name: '前端开发' }
var b = a;
b.name = '进阶';

// 这时a.name的值是多少
```

&emsp;&emsp;答： `a.name` 的值是：<font color="white"> 进阶 </font>。

&emsp;&emsp;问题中 `a` 是引用类型，它在栈中保存了一个对象的地址，`b` 对 `a` 进行复制的时候，复制的就是这个对象的地址，所以它们指向的对象是同一个对象，因此 `b` 的修改也会反映到 `a` 中。



**问题2**

```js
var a = { name: '前端开发' }
var b = a;
a = null;

// 这时b的值是多少
```

&emsp;&emsp;答：`b` 的值是：<font color="white"> {name: '前端开发'} </font> 。

&emsp;&emsp;同样的，`b` 对 `a` 的复制只是复制了指向堆中的对象的内存地址。然后 `a` 被修改为 `null`，也就是把 `a` 存储在栈中的内存地址变成了基本类型 `null`，并不影响 `b` 和堆内存中的对象，因此 `b` 还是该对象。



**问题3**

```js
var a = {n: 1};
var b = a;
a.x = a = {n: 2};

a.x 	// 这时 a.x 的值是多少
b.x 	// 这时 b.x 的值是多少
```

&emsp;&emsp;答：`a.x = ` <font color="white">undefined</font>，`b.x = ` <font color="white">{n : 2}</font> （鼠标选中空白处查看答案）

&emsp;&emsp;这道题的关键在于：

1. 运算符的优先级：`.` 运算符的优先级高于 `=`，因此先执行 `a.x`，堆中内存的 `{n: 1}` 就变成了 `{n: 1，x: undefined}`，`b` 和 `a` 的指向是相同的，因此 `b.x` 也发生了变化。
2. 运算符的结合性：**赋值操作**的结合性是**从右到左**的，因此会先执行 `a = {n: 2}`，`a` 的引用就被改变了，但是在这一步中 `a.x` 还是第一步中的 `{n: 1, x: undefined} ` 对象，也就相当于 `b.x`，因此就是 `b.x = {n: 2}`。



&emsp;&emsp;**但是**，**闭包**中的变量并不保存在栈内存中，而是**保存在堆中**，这也就解释了为什么函数使用完后闭包还能引用到函数内的变量。

```js
function A() {
    let a = 1
    function B() {
        console.log(a)
    }
    return B
}
```

&emsp;&emsp;上面代码中，函数 `B` 就被称为闭包。

&emsp;&emsp;函数 A 弹出调用栈后，函数 A 中的变量是存储在堆上的，因此函数 B 依旧能够引用到函数 A 中你的变量。JS 引擎可以通过逃逸分析辨别出哪些变量需要存储在堆上，哪些变量需要存储在栈中。



**问题4** ：从内存来看， `null` 和 `undefined` 的区别是什么？

&emsp;&emsp;答：给一个**全局变量**赋值为 `null`，相当于将这个变量的指针对象及值清空；如果是给**对象的属性**或**局部变量**赋值为 `null`，相当于给这个属性分配了一块空的内存，然后值为 `null`，JS 会回收全局变量为 `null` 的对象。

&emsp;&emsp;给一个全局变量赋值为 `undefined` ，相当于将这个对象的值清空，但是这个对象依然存在；如果是给对象的属性赋值为 `undefined`，说明这个值为空值。

# 内存回收

&emsp;&emsp;JS 有自动垃圾回收机制，垃圾收集器每隔一段时间就执行一次，找出那些不再继续使用的值，然后释放其占用的内存。

- 局部变量和全局变量的回收
  - **局部变量**：局部作用域中，当函数执行完毕，局部变量也就没有存在的必要了，因此垃圾收集器很容易做出判断并回收
  - **全局变量**：全局变量什么时候需要自动释放内存空间很难判断，因此在开发中尽量避免使用全局变量。
- 以 V8 为例，V8 中所有的 JS 对象都是通过**堆**来分配内存的：
  - **初始分配**：当变量声明并赋值时，V8 就会在堆中给这个遍历分配内存。
  - **继续申请**：当已申请的内存不足以存储这个变量时，V8 就会继续申请内存，直到堆的大小达到了 V8 内存上限为止。
- V8 对堆内存中的对象进行**分代**管理
  - **新生代**：存活周期较短的 JS 对象，如临时变量、字符串等。
  - **老生代**：经过多次垃圾回收仍存活、存活周期较长的对象，如主控制器、服务器对象等。

# 垃圾回收算法

&emsp;&emsp;常用的垃圾回收算法有两种：

1. **引用计数**（现代浏览器已不再使用）
2. **标记清除**（常用）

## 引用计数

&emsp;&emsp;引用计数的内存使用判断标准很简单，就是看一个对象是否有指向它的引用，若没有其他对象指向它，说明这个对象不会再被使用到了。

```js
// 创建一个对象person，他有两个指向属性age和name的引用
var person = {
    age: 12,
    name: 'aaaa'
};

person.name = null; // 虽然name设置为null，但因为person对象还有指向name的引用，因此name不会回收

var p = person; 
person = 1;    //原来的person对象被赋值为1，但因为有新引用p指向原person对象，因此它不会被回收

p = null;   //原person对象已经没有引用，很快会被回收
```

&emsp;&emsp;引用计数有一个致命的问题，那就是**循环引用**。

&emsp;&emsp;如果两个对象相互引用，尽管他们已不再使用，但垃圾回收器不会进行回收，最终可能会导致**内存泄露**。如：

```js
function cycle() {
	var o1 = {};
    var o2 = {};
    o1.a = o2;
    o2.a = o1;
    
    return 'cycle reference';
}

cycle();
```

&emsp;&emsp;上面的代码中，`cycle` 函数执行完后，对象 `o1` 和 `o2` 实际上已经不再需要了，但是根据引用计数规则，它们之间的相互引用仍然存在，因此这部分内存不会被回收。

## 标记清除

&emsp;&emsp;**标记清除（Mark-and-sweep）** 算法将 “不再使用的对象” 定义为 “**无法到达的对象**”。即从根部（JS中是全局对象）出发定时扫描内存中的对象，**凡是能从根部到达的对象都保留**，无法从根部到达的对象被标记为不再使用，稍后会进行回收。

&emsp;&emsp;标记清除算法由以下几步组成：

1. 垃圾回收器创建一个 `roots` 列表。`roots` 通常是代码中全局变量的引用。JS 中， `window` 对象是一个 `root`，因为 `window` 对象总是存在，因此垃圾回收器可以检查它和它的所有子对象是否存在（即不是垃圾）；
2. 所有的 `roots` 被检查和标记为激活（不是垃圾）。所有的子对象也被递归地检查。从 `root` 开始的所有对象如果是可达的，它就不会被当做垃圾。
3. 所有未被标记的内存会被当做垃圾，收集器会释放它们的内存，归还给操作系统。



# 内存泄漏

[阮一峰内存泄漏教程](https://ruanyifeng.com/blog/2017/04/memory-leak.html)



&emsp;&emsp;对于持续运行的服务进程（daemon），必须及时释放不再用到的内存。否则，内存占用越来越高，轻则影响系统性能，重则导致进程崩溃。

&emsp;&emsp;**对于不再用到的内存，没有及时释放，就叫做内存泄漏（memory leak）**

## 内存泄漏的识别方法

&emsp;&emsp;[经验法则](https://www.toptal.com/nodejs/debugging-memory-leaks-node-js-applications)是，如果连续五次垃圾回收之后，内存占用一次比一次大，就有内存泄漏。这就要求实时查看内存占用。

### 1 浏览器方法

1. 打开开发者工具，选择 Memory
2. 在右侧的 Select profiling type 字段里勾选 timeline
3. 点击左上角的录制按钮
4. 在页面上进行各种操作，模拟正常的使用情况
5. 一段时间后，点击左上角的 stop 按钮，面板上就会显示这段时间的内存占用情况

### 2 命令行方法

&emsp;&emsp;使用 Node 提供的 `process.memoryUsage` 方法。

```js
console.log(process.memoryUsage());

// 输出
{ 
  rss: 27709440,		// resident set size，所有内存占用，包括指令区和堆栈
  heapTotal: 5685248,   // "堆"占用的内存，包括用到的和没用到的
  heapUsed: 3449392,	// 用到的堆的部分
  external: 8772 		// V8 引擎内部的 C++ 对象占用的内存
}
```

&emsp;&emsp;判断内存泄漏，以`heapUsed`字段为准。



&emsp;&emsp;ES6 新增的两种数据结构 [WeakSet 和 WeakMap](https://yleave.top/2020/12/09/JS/Set%E3%80%81Map%E3%80%81%E5%92%8CWeakSet%E3%80%81WeakMap/#weakset) ，使用了一种弱引用机制，可以有效解决常见的 DOM 节点泄漏问题。

# 四种常见的 JS 内存泄漏

## 1.意外的全局变量

&emsp;&emsp;未定义的变量会在全局对象下创建一个新变量，如：

```js
function foo(arg) {
    bar = "this is a hidden global variable"; // window.bar = "..."
}
```

&emsp;&emsp;函数 `foo` 内未使用 `var` 等关键字声明变量，JS 会将其挂载到全局对象上，意外的创建一个全局变量。

&emsp;&emsp;另一个意外的全局变量可能由 `this` 创建：

```js
function foo() {
    this.variable = "potentail accidental global";
}

// 此时 foo 中的 this 指向了全局对象，因此 this.variable 变成了全局变量
foo();
```

&emsp;&emsp;**解决方法**：在 JS 文件头部加上 `use strict`，使用严格模式避免意外的全局变量，此时上例中的 `this` 指向 `undefined`。

&emsp;&emsp;若需要使用全局变量存储大量数据时，确保用完后将它设置为 `null` 或重新定义以保证能够被回收。

## 2. 被遗忘的计时器或回调函数

计时器 `setInterval`：

```js
var someResource = getData();
setInterval(function() {
    var node = document.getElementById('Node');
    if(node) {
        // 处理 node 和 someResource
        node.innerHTML = JSON.stringify(someResource));
    }
}, 1000);
```

&emsp;&emsp;在上面的例子中，在节点 `node` 不再需要时，定时器仍然指向这些数据，因此即使 `node` 节点被移除， `interval` 仍然存活且垃圾回收器没办法回收，它的依赖也没办法被回收，除非终止定时器。

## 3. 脱离 DOM 的引用

&emsp;&emsp;如果把 DOM 保存到字典或数组中，这样 DOM 元素会存在两个引用：一个在 DOM 树中，另一个在字典中，那么当不需要的时候需要把这两个引用都清除：

```js
var elements = {
    button: document.getElementById('button'),
    image: document.getElementById('image'),
    text: document.getElementById('text')
};
function doStuff() {
    image.src = 'http://some.url/image';
    button.click();
    console.log(text.innerHTML);
    // 更多逻辑
}
function removeButton() {
    // 按钮是 body 的后代元素
    document.body.removeChild(document.getElementById('button'));
    // 此时，仍旧存在一个全局的 #button 的引用
    // elements 字典。button 元素仍旧在内存中，不能被 GC 回收。
}
```

&emsp;&emsp;此外，**如果代码中保存了表格中某一个 `<td>` 的引用**，将来决定删除表格后，直觉会认为 GC 会回收除了已保存的 `<td>` 之外的其他节点。

&emsp;&emsp;但实际情况并非如此：**这个 `<td>` 是表格的子节点，子元素与父元素是引用关系**。由于代码中保存了子元素 `<td>` 的引用，导致整个表格仍保存在内存中。

## 4. 闭包

&emsp;&emsp;闭包的关键是匿名函数可以访问父级作用域的变量。

```js
var theThing = null;
var replaceThing = function () {
    var originalThing = theThing;
    
    var unused = function () {
        if (originalThing)
            console.log("hi");
    };

    theThing = {
        longStr: new Array(1000000).join('*'),
        someMethod: function () {
            console.log(someMessage);
        }
    };
};

setInterval(replaceThing, 1000);
```

&emsp;&emsp;在上面的代码中，每次调用 `replaceThing`，`theThing` 得到一个包含一个大数组和一个新闭包（`someMethod`）的新对象。同时，变量 `unused` 是一个引用 `originalThing` 的闭包。

&emsp;&emsp;`someMethod` 可以通过 `theThing` 使用，`someMethod` 与 `unused` 分享闭包的作用域，尽管 `unused` 从未使用，它引用的 `originalThing` 迫使它保留在内存中（防止被回收）。

&emsp;&emsp;**解决方法**：在 `replaceThing` 函数的最后面添加 `originalThing = null`。 



---

REF: 

https://muyiy.cn/blog/1/1.3.html

https://muyiy.cn/blog/1/1.4.html

https://muyiy.cn/blog/1/1.5.html

