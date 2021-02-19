---
title: JS 对象
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/js.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/63.jpg'
date: 2020-11-27 16:16:26
categories:
    - JS
tags:
    - JS Object
---



# 对象

## **JS 中，对象的 `key` 都会被转为 `string` 类型。**

&emsp;&emsp;**如：**

```js
const obj = { 1: 'a', 2: 'b', 3: 'c' };
obj.hasOwnProperty('1');  // true
obj.hasOwnProperty(1);  // true
```

&emsp;&emsp;**再如**，以一个对象为 `key`，同样会被转为 `string`：

```js
const a = {};
const b = { key: 'b' };
const c = { };

a[b] = 123;
a[c] = 456;

console.log(a[b]); // 456
```

&emsp;&emsp;对象 `a`：<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200416203843666.png" alt="image-20200416203843666" style="zoom:80%;" />

&emsp;&emsp;上面的代码中， 对象 `b` 和对象`c` 都被当做一个 `key`，而对象的 `key` 会被转为字符串，因此 `b` 和 `c` 会被转为：`"[object Object]"`，所以，代码中表示 `a["object Object"] = 123`，所以最后变成了 `456`。

&emsp;&emsp;当我们调用 `a[b]` 的时候，相当于调用了 `a["object Object"]`。

&emsp;&emsp;不过若是用 `a.b` 的话，`b` 就是代表一个字符 `b` 。



## 判断对象是否为空

### 1. object.keys()

```js
var data = {};
var arr = Object.keys(data);
alert(arr.length == 0); //true 为空， false 不为空
```

### 2.将对象转化为json字符串，再判断该字符串是否为"{}"

```js
var data = {};
var b = (JSON.stringify(data) == "{}");
alert(b);   //true 为空， false 不为空
```

### 3.for in 循环判断

```js
var obj = {};
var b = function() {
    for(var key in obj) {
        return false;
    }
    return true;
}
```

### 4.Object.getOwnPropertyNames()方法

&emsp;&emsp;此方法是使用 `Object` 对象的 `getOwnPropertyNames` 方法，获取到对象中的属性名，存到一个数组中，返回数组对象，我们可以通过判断数组的 `length ` 来判断此对象是否为空。

```js
var data = {};
var arr = Object.getOwnPropertyNames(data);
alert(arr.length == 0);//true
```



## 比较对象中的内容是否相同，编写 Object.equals 函数

&emsp;&emsp;注意，在 `equals` 中没有使用 `===` 进行比较，因为 `===` 会比较对象的引用是否相同，而我们只想比较内容是否相同：

```js
Object.prototype.equals = function(obj) {
    // For first loop , we only check for types
    for (let propName in this) {
        // Check for inherited methods and properties
        if (this.hasOwnProperty(propName) != obj.hasOwnProperty(propName)) {
            return false;
        }

        // Check instance type
        if (typeof this[propName] != typeof obj[propName]) {
            return false;
        }
    }

    // A deeper check using other objects property names
    for (let propName in obj) {
        // We must check instances anyway, there may be a property exits in obj
        if (this.hasOwnProperty(propName) != obj.hasOwnProperty(propName)) {
            return false;
        }

        if (typeof this[propName] != typeof obj[propName]) {
            return false;
        }

        // If the property is inherited, do not check any more (it must be equal if both objects inherit it)
        if (!this.hasOwnProperty(propName)) {
            continue;
        }

        // Now the detail check and recursion

        // Requires Array.equals !!!
        if (this[propName] instanceof Array && obj[propName] instanceof Array) {
            // recurse into the nested arrays
            if (!this[propName].equals(obj[propName])) {
                return false;
            }
        } else if (this[propName] instanceof Object && obj[propName] instanceof Object) {
            // recurse into other objects
            // console.log("Recursing to compare ", this[propName],"with",obj[propName], " both named \""+propName+"\"");
            if (!this[propName].equals(obj[propName])) {
                return false;
            }
        } else if (this[propName] != obj[propName]) {  // Normal value comparsion for strings and numbers
            return false;
        }
    }
    return true;
}
```

&emsp;&emsp;一个 [Array,prototype.equals](https://yleave.top/2020/11/27/JS/JS-%E6%95%B0%E7%BB%84/#%E5%88%A4%E6%96%AD%E4%B8%A4%E4%B8%AA%E6%95%B0%E7%BB%84%E7%9A%84%E5%86%85%E5%AE%B9%E6%98%AF%E5%90%A6%E7%9B%B8%E5%90%8C) 的实现

&emsp;&emsp;上面代码中还需要实现 `Array.prototype.equals` 方法，这边模拟时就先都返回 `true` ：

```js
Array.prototype.equals = function(arr) {
    return true;
};

let obj1 = {
    name: 'yy',
    age: '15',
    objs: {
        obj1: '教师',
        obj2s: ['程序员', '码农'],
        obj3s: {
            obj3_1: '互联网员工',
            obj3_2: '社畜'
        }
    }
};

let obj2 = {
    name: 'yy',
    age: '15',
    objs: {
        obj1: '教师',
        obj2s: ['程序员', '码农'],
        obj3s: {
            obj3_1: '互联网员工',
            obj3_2: '社畜'
        }
    }
};

console.log(obj1.equals(obj2)); // true
```









## 对象的一些方法

### Object.prototype.hasOwnProperty()

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/hasOwnProperty)

&emsp;&emsp;`hasOwnProperty` 方法可以检测某个属性是否是该对象自身的（不包括原型链上的属性），若是，返回 `true` ，否则返回 `false`。

例：

```js
function Obj() {
    this.name = 'obj';
}

Obj.prototype = {
    age: 12
};

Object.prototype.job = 'new Job';

let o = new Obj();

for (let v in o) {
    console.log(v, o.hasOwnProperty(v));
}
```

&emsp;&emsp;程序输出：

```
name true
age false
job false
```

### Object.defineProperty()

&emsp;&emsp;`Object.defineProperty()` 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性， 并**返回这个对象**。

&emsp;&emsp;语法：`Object.defineProperty(obj, prop, descriptor)`

- `obj` : 要在其上定义属性的对象。
- `prop` : 要定义或修改的属性的名称。
- `descriptor` : 将被定义或修改的属性描述符。

例：

```js
const person = { name: "Lydia" };

Object.defineProperty(person, "age", { value: 21 });

console.log(person); // { name: "Lydia", age: 21 }
console.log(Object.keys(person)); // ["name"]
```

&emsp;&emsp;可修改 `descriptor` 来设置添加的属性。

- `configurable` :  默认为 `false`，当设置为 `true` 时，`descriptor` 才可被修改，同时该属性也能从对应的对象上被删除。
- `enumerabel` :  默认为 `false`，设置为 `true` 时，新添加的属性才能被枚举
- `value` :  默认为 `undefined`，可以是任何有效的 JS 值（数值，对象，函数等）
- `writable` :  默认为 `false`，设置为 `true` 时，`value` 才能被赋值运算符修改。
- `get` :  默认为 `undefined`，一个给属性提供 `getter` 的方法，当访问该属性时，该方法会被执行，方法执行时没有参数传入，但是会传入 `this` 对象
- `set` :  默认为 `undefined`，一个给属性提供 `setter` 的方法，当属性值修改时，触发执行该方法。该方法将接受唯一参数，即该属性新的参数值。



### Object.entries

&emsp;&emsp;`Object.entries(obj)` 方法返回一个给定对象自身**可枚举属性的键值对数组**，其排列与使用 `for...in` 循环遍历该对象时返回的顺序一致，即属性的顺序与通过手动循环对象的属性值所给出的顺序相同。（**区别在于 `for-in` 循环还会枚举原型链中的属性**）。

例：

```js
const anObj = { 100: 'a', 2: 'b', 7: 'c' };
console.log(Object.entries(anObj)); // [ ['2', 'b'], ['7', 'c'], ['100', 'a'] ]

// 字符串
console.log(Object.entries('foo')); // [ ['0', 'f'], ['1', 'o'], ['2', 'o'] ]
```

&emsp;&emsp;使用 `for ... of` 和解构赋值遍历：

```js
// 使用 for - of 和 解构赋值 遍历结果
const obj = { a: 5, b: 7, c: 9 };
for (const [key, value] of Object.entries(obj)) {
    console.log(`${key} ${value}`); // "a 5", "b 7", "c 9"
}
```

&emsp;&emsp;使用 `forEach` 遍历：

```js
Object.entries(obj).forEach(([key, value]) => {
    console.log(`${key} ${value}`); // "a 5", "b 7", "c 9"
});
```

&emsp;&emsp;`new Map()` 构造函数接受一个可迭代的 `entries`。借助 `Object.entries` 方法你可以很容易的将 `Object` 转换为 `Map`:

```js
var obj = { foo: "bar", baz: 42 }; 
var map = new Map(Object.entries(obj));
console.log(map); // Map { foo: "bar", baz: 42 }
```

### Object.assign()

[MDN介绍](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

&emsp;&emsp;`Object.assign()` 方法用于将所有**可枚举属性**的值从一个或多个源对象复制到目标对象。它将**返回目标对象**。

&emsp;&emsp;如果目标对象中的属性具有**相同的键**，则属性将被源对象中的属性**覆盖**。后面的源对象的属性将类似地覆盖前面的源对象的属性。

&emsp;&emsp;语法：`Object.assign(target, ...sources)`

- `target `: 目标对象。
- `sources `: 源对象。

返回值：目标对象。

&emsp;&emsp;==注意==，`Object.assign` 不会在那些`source`对象值为 [null](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/null) 或 [undefined](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/undefined) 的时候抛出错误。



例：

```js
const target = { a: 1, b: 2 };
const source = { b: 4, c: 5 };

const returnedTarget = Object.assign(target, source);

console.log(target); // Object { a: 1, b: 4, c: 5 }
console.log(returnedTarget); //Object { a: 1, b: 4, c: 5 }
```

&emsp;&emsp;**复制一个对象：**

```js
const obj = { a: 1 };
const copy = Object.assign({}, obj);
console.log(copy); // { a: 1 }
```



&emsp;&emsp;针对**深拷贝**，需要使用其他办法，因为  `Object.assign() `拷贝的是属性值。**假如源对象的属性值是一个对象的引用，那么它也只指向那个引用**。

```js
let obj1 = { a: 0 , b: { c: 0}}; 
let obj2 = Object.assign({}, obj1); 
console.log(JSON.stringify(obj2)); // { a: 0, b: { c: 0}} 

obj1.a = 1; 
console.log(JSON.stringify(obj1)); // { a: 1, b: { c: 0}} 
console.log(JSON.stringify(obj2)); // { a: 0, b: { c: 0}} 

obj2.a = 2; 
console.log(JSON.stringify(obj1)); // { a: 1, b: { c: 0}} 
console.log(JSON.stringify(obj2)); // { a: 2, b: { c: 0}}
 
// 改变源对象中的对象属性值，拷贝对象会跟着改变，因为只是拷贝了对象的引用
obj2.b.c = 3; 
console.log(JSON.stringify(obj1)); // { a: 1, b: { c: 3}} 
console.log(JSON.stringify(obj2)); // { a: 2, b: { c: 3}} 
```



&emsp;&emsp;**深克隆：**

```js
// Deep Clone 
obj1 = { a: 0 , b: { c: 0}}; 
let obj3 = JSON.parse(JSON.stringify(obj1)); 
obj1.a = 4; 
obj1.b.c = 4; 
console.log(JSON.stringify(obj3)); // { a: 0, b: { c: 0}}
```











