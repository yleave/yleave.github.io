---
title: 2.JS执行上下文和变量对象
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/js.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/10.jpg'
date: 2021-01-01 20:40:14
categories:
    - JS
    - JS调用堆栈
tags:
    - 执行上下文
    - 执行栈
    - 变量对象
---



&emsp;&emsp;JS 是单线程语言，因此执行顺序是**顺序执行**，不过 JS 引擎在执行 JS 代码的时候并不是逐行执行，而是一段一段地分析执行，先是编译阶段，然后才是执行阶段。



&emsp;&emsp;具体的体现可看例子：

**例一  变量提升**

&emsp;&emsp;我们在未定义 `foo` 之前就使用了它，结果不会报错，而是会为 `undefined`，随后， `foo` 会像我们定义的那样先是输出 `foo1` ，后输出 `foo2`。

```js
console.log(foo);   // undefined

var foo = function() {
    console.log('foo1');
}

foo();  // foo1

var foo = function() {
    console.log('foo2');
}

foo();  // foo2
```

**例二  函数提升**

&emsp;&emsp;在定义之前就调用 `foo` ，不会发生错误，且后定义的 `foo` 会覆盖先定义的同名函数。

```js
foo();  // foo2

function foo() {
    console.log('foo1');
}

foo();  // foo2

function foo() {
    console.log('foo2');
}

foo();  // foo2
```

**例三  声明优先级：函数 > 变量**

&emsp;&emsp;首先，函数提升的优先级会高于变量提升，因此开始输出 `foo2`，然后 `foo` 被重新定义，接下来都输出 `foo1`。

```js
foo();  // foo2

var foo = function() {
    console.log('foo1');
}

foo();  // foo1

function foo() {
    console.log('foo2');
}

foo();  // foo1
```

# 执行上下文栈

&emsp;&emsp;JS 使用了执行上下文栈（Execution context stack，ESC）来管理执行上下文。

&emsp;&emsp;这个在 [1-JS执行上下文和执行栈](https://yleave.top/2020/12/29/JS/JS%E8%B0%83%E7%94%A8%E5%A0%86%E6%A0%88/1.JS-%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87%E5%92%8C%E6%89%A7%E8%A1%8C%E6%A0%88/) 中就有提到



&emsp;&emsp;关于执行上下文栈，有以下例子：

**代码1**

```js
var scope = 'global scope';

function checkScope() {
    var scope = 'local scope';

    function f() {
        return scope;
    }

    return f();
}

console.log(checkScope());
```

**代码2**

```js
var scope = 'global scope';

function checkScope() {
    var scope = 'local scope';

    function f() {
        return scope;
    }

    return f();
}

console.log(checkScope()());
```

&emsp;&emsp;这两段代码的输出都为  <font color="white">local scope</font>。(选中看答案)

&emsp;&emsp;原因很简单，是因为 JS 采用的是词法作用域，函数的作用域基于函数创建的位置。

&emsp;&emsp;用 《JS 权威指南》的回答就是：

> JS 函数的执行用到了作用域链，这个作用域链是在函数定义的时候创建的。
>
> 嵌套的函数 `f()` 定义在这个作用域链里，其中的变量 `scope` 一定是局部变量，不管何时何地执行函数 `f()` 这种绑定在执行 `f()` 时依然有效。

&emsp;&emsp;**而这两段代码的不同之处在于其执行上下栈的变化不一样：**

&emsp;&emsp;使用伪代码表示，**第一段**代码的执行上下文栈是这样的：

```js
ECStack.push(<checkscope> functionContext);
ECStack.push(<f> functionContext);
ECStack.pop();
ECStack.pop();
```

&emsp;&emsp;而**第二段**代码为：

```js
ECStack.push(<checkscope> functionContext);
ECStack.pop();
ECStack.push(<f> functionContext);
ECStack.pop();
```

# 函数上下文

## 变量对象

&emsp;&emsp;**变量对象(variable object，VO)**是与执行上下文相关的数据作用域，存储了在上下文中定义的变量和函数声明。



&emsp;&emsp;在函数上下文中，使用**活动对象（activation object，AO）**来表示**变量对象**。

&emsp;&emsp;变量对象和活动对象实际上是一个东西，它们的区别在于：

- 变量对象（**VO**）是规范上或是 JS 引擎上实现的，并不能在 JS 环境中直接访问
- 当进入到一个执行上下文后，这个变量对象才会被激活，所以叫活动对象（**AO**），这时候活动对象上的各种属性才能被访问。

&emsp;&emsp;在调用函数时，会为其创建一个 `Arguments` 对象，并自动初始化剧本变量 `arugments`，指代 `Arguments` 对象。这个对象中存储了所有传入函数的参数。

&emsp;&emsp;因此 **AO = VO + function parameters + arguments**

# 执行过程

&emsp;&emsp;执行上下文中的代码会分为两个阶段进行处理：

1. 进入执行上下文
2. 代码执行

## 进入执行上下文

&emsp;&emsp;此时的**变量对象**会以以下顺序初始化：

1. **函数的所有形参**（函数上下文中）：

   - 由名称和对应值组成的一个变量对象的属性被创建

   - 没有实参，属性值设为 `undefined`

2. **函数声明**：

   - 由名称和对应值（function-object）组成的一个变量对象的属性被创建

   - 如果变量对象已经存在相同名称的属性，则完全替换这个属性。

3. **变量声明**：

   - 由名称和对应值（`undefined`）组成一个变量对象的属性被创建

   - 如果变量名称跟已经声明的形参或函数相同，则遍历声明不会干扰已存在的这类属性。

如下代码：

```js
function foo(a) {
    var b = 2;
    function c() {}
    var d = function() {};
    b = 3;
}

foo(1);
```

&emsp;&emsp;进入执行上下文后，这个时候的（活动对象） AO 是：

```js
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: undefined,
    c: reference to function c() {},
    d: undefined
}
```

&emsp;&emsp;形参 `a` 和 `arguments` 这时候已经有赋值了，而变量还是 `undefined`。

## 代码执行

&emsp;&emsp;这个阶段会顺序执行代码，并修改变量对象的值，执行完成后 `AO` 如下：

```js
AO = {
    arguments: {
        0: 1,
        length: 1
    },
    a: 1,
    b: 3,
    c: reference to function c() {},
    d: reference to FunctionExpression "d"
}
```



## 小结

1. 全局上下文的变量对象初始化是全局对象
2. 函数上下文的变量对象初始化只包括 `Arguments` 对象
3. 在进入执行上下文时会给变量对象添加形参、函数声明、变量声明等初始的属性值
4. 在代码执行阶段，会再次修改变量对象的属性值



## 两个例子

### 1

```js
function foo() {
    console.log(a);
    a = 1;
}

foo();
```

&emsp;&emsp;上面的代码会报错：`Uncaught ReferenceError: a is not defined`

&emsp;&emsp;这是因为函数中的 `a` 没有通过 `var` 关键字声明，所以不会被存放在 `AO` 中。

&emsp;&emsp;在执行 `console.log` 的时候，`AO` 的值是：

```js
AO = {
    arguments: {
        length: 0
    }
}
```

&emsp;&emsp;没有 `a` 的值，然后就去全局对象中寻找，也没找到，因此报错了。

&emsp;&emsp;如果在函数中使用了 `var` 声明的话：

```js
console.log(a)
var a = 1;
```

&emsp;&emsp;那么 `console` 会打印 `undefined`。

### 2

```js
console.log(foo);

function foo() {
    console.log('foo');
}

var foo = 1;
```

&emsp;&emsp;上面的代码会打印函数，而不是 `undefined`。

&emsp;&emsp;这是因为在进入执行上下文时，首先会处理函数声明，其次处理变量声明，如果变量名称跟已经声明的形参或函数相同，则变量声明不会干扰已存在的这类属性。

---

REF：https://muyiy.cn/blog/1/1.2.html#%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87