---
title: 手撕内核源码
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/眼镜.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/44.png'
date: 2020-12-13 22:16:40
categories:
    - 前端面试题
tags:
    - 手撕代码
---

# 手写 Array.prototype.map

## map 概念

`map(callback(val, idx, arr), thisArg)` 方法将创建一个新数组，这个数组中的元素是原数组中的每个元素都调用 `callback` 后的结果，其中 `callback` 的三个参数分别是原数组中的元素、元素对应索引值和原数组，`thisArg` 是 `map` 函数的 `this` 指向。

因此调用 `map` 函数后，原数组不会发生改变。

且，调用的数组 `arr` 中的元素不一定是连续的（有的索引位置会为 `empty`），这点需要注意。

```js
if (!Array.prototype.map) {
    Array.prototype.map = function(callback, thisArg) {
        let A, T, k;

        if (this == null) {
            throw new TypeError("this is null or undefined!");
        }

        // 1. 将 O 赋值为调用 map 方法的数组.
        // 使用 Object 是为了保证 o 为对象
        let O = Object(this);

        // 2. 将 len 赋值为数组 O 的长度.  
        // 这边 <<< 运算符为 零填充右移运算符，如 0101 >>> 1 : 0010，猜测这么做是为了防止 len 为字符串
        let len = O.length >>> 0;

        // 3. 如果 callback 不是函数,则抛出 TypeError 异常.
        if (Object.prototype.toString.call(callback) != "[object Function]") {
            throw new TypeError(callback + "is not a function!");
        }

        // 4. 如果参数 thisArg 有值,则将 T 赋值为 thisArg; 否则T为 undefined.
        if (thisArg) {
            T = thisArg;
        }

        // 5. 创建新数组 A,长度为原数组 O 长度 len
        A = new Array(len);

        // 6. 将 k 赋值为0
        k = 0;

        // 7. 当 k < len 时,执行循环.
        while (k < len) {

            let kVal, mappedVal;

            //遍历 O, k 为原数组索引 , 这边有个疑问，为什么不直接遍历 O，因为如果数组中的元素很松散的话 k++ 效率可能会很低
            if (k in O) {

                // kVa为索引 k 对应的值.
                kVal = O[k];

                // 执行 callback,this 指向 T,参数有三个.分别是kVal:值, k:索引, O:原数组.
                mappedVal = callback.call(T, kVal, k, O);

                // 返回值添加到新数组 A 中.
                A[k] = mappedVal;
            }

            k++;
        }

        return A;
    }
}
```



上面代码中的 `1`：使用 [Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)  是为了保证 `o` 一定是一个对象：

- 当给定值是 `null` 或 `undefined` 时，会创建并返回一个空对象
- 若传进去的是一个基本类型的值，则会构造其包装类型的对象，如 `Object(3)` ，会返回 `Number {3}`
- 若传的是引用类型的值，仍会返回这个值，因此引用是相同的



# 数组扁平化

## flat 函数

 `flat `[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/flat) 

> `flat(deep)` 方法会根据指定的递归深度遍历数组，并将遍历到的元素合并为一个**新数组**返回

```js
const test = ["a", ["b", "c"], ["d", ["e", ["f"]], "g"]]
```

**`flag` 不传参数时，默认扁平化一层**

```js
test.flat()
// ["a", "b", "c", "d", ["e", ["f"]], "g"]
```

**传入的参数即扁平化的深度**

```js
test.flat(2)
// ["a", "b", "c", "d", "e", ["f"], "g"]
```

**当使用 `Infinity` 作为参数时，无论多少层嵌套，都会扁平化为一维数组**

```js
test.flat(Infinity)
// ["a", "b", "c", "d", "e", "f", "g"]
```

**传入小于等于 `0` 的参数，不进行扁平化**

```js
test.flat(0)
test.flat(-1)
// ["a", ["b", "c"], ["d", ["e", ["f"]], "g"]]
```

**若数组不是连续的，会跳过那些空位**

```js
["a", , "b", "c", ,].flat()
// ["a", "b", "c"]
```



## 手写扁平化函数

### 1 使用 reduce 方法

一次性扁平化所有：

```js
function flattenDeep(arr) {
    return Array.isArray(arr) ?
        arr.reduce((acc, cur) => [...acc, ...flattenDeep(cur)], [])
    	: [arr];
}
```

在 `flattenDeep` 的基础上实现 `flat` ：

```js
if (!Array.prototype.flat) {
    Array.prototype.flat = function(deep=1) {
        return deep > 0 ?
            this.reduce((acc, cur) => {
                if (Array.isArray(cur)) {	// 若是数组且 deep 大于 0，继续递归
                    return [...acc, ...cur.flat(deep-1)];
                }
                return [...acc, cur];	// 否则将当前值加入累加器中
            }, [])
            : this;
    };
}
```

### 2 使用栈

先将 `arr` 中的数据存入栈 `stack` 中，然后从后到前，遍历栈中的每一个元素，若是数组，则将其展开一层再压入栈中，若不是数组则使用 `unshift` 存入结果数组中。



一次性扁平化所有：

```js
function flattenDeep(arr) {
    let result = [];

    let stack = [...arr];

    while (stack.length) {
        let val = stack.pop();

        if (Array.isArray(val)) {
            stack.push(...val);
        } else if (val) {   // 去除空位
            result.unshift(val);
        }
    }

    return result;
}
```

根据 `deep` 扁平化：

```js
function flattenDeep(arr, deep=1) {
    let result = [];

    let stack = [...arr];

    while (stack.length) {
        let val = stack.pop();

        if (Array.isArray(val) && deep > 0) {
            result.unshift(...flattenDeep(val, deep-1));
        } else if (val) {   // 去除空位
            result.unshift(val);
        }
    }

    return result;
}
```



# Array.prototype.equals

不能使用 `===` 来判断，因为数组也是对象，而对象会根据其引用来判断是否相等。



```js
const arr1 = ["a", , ["b", "c"], ["d", ["e", ["f"]], "g"], {'h': 1, 'i': 2}]
const arr2 = ["a", ["d", ["e", ["f"]], "g"], , ["b", "c"], {'h': 1, 'i': 2}]
```

## 先转为字符串，再判断字符串是否相等

**若进行了排序，则表示不考虑数组顺序**

字符串的转换方法有：`arr.toString()` 和 `JSON.stringfy(arr)` 和 `arr.join()`

在转为字符串前需要对数组进行排序

```js
function isEqual(arr1, arr2) {
    return JSON.stringify(arr1.sort()) == JSON.stringify(arr2.sort());
}
```



## 手写 equals 方法

若要**将数组顺序考虑在内**，可使用以下方法，不过下面没有将数组元素是 `object` 实例的情况：

```js
if (!Array.prototype.equals) {
    Array.prototype.equals = function(array) {
        // 若 array 是虚值，直接返回
        if (!array) {
            return false;
        }

        // 先判断数组长度是否相等，若不相等返回 false
        if (this.length != array.length) {
            return false;
        }

        for (let i = 0, l = this.length; i < l; ++i) {
            // 判断是否有循环嵌套
            if (this[i] instanceof Array && array[i] instanceof Array) {
                if (!this[i].equals(array[i])) {
                    return false;
                }
            } else if (this[i] != array[i]) {
                return false;
            }
            // 这边没有考虑数组元素是 object 的情况
        }

        return true;
    };
}
```



# Object.prototype.euqals

## 手写 equals 方法

```js
Object.prototype.equals = function(obj) {
    // 第一次循环，检查 this 中的属性名和属性值类别是否相同
    for (let propName in this) {
        if (this.hasOwnProperty(propName) != obj.hasOwnProperty(propName)) {
            return false;
        } else if (typeof this[propName] != typeof obj[propName]) {
            return false;
        }
    }

    // 第二次循环，检查 obj 中的属性名和属性值类别是否和 this 中的相同
    // 并递归进行检查
    for (let propName in obj) {
        // 因为可能有的属性只存在与 obj 中
        if (this.hasOwnProperty(propName) != obj.hasOwnProperty(propName)) {
            return false;
        } else if (typeof this[propName] != typeof obj[propName]) {
            return false;
        }

        // 若该属性是继承自原型链的，那么肯定相等，不需要检查
        if (!this.hasOwnProperty(propName)) {
            continue;
        }

        // 进行递归检查

        // 首先检查是否是一个数组类型，需要实现数组的检查方法 Array.prototype.equals
        if (this[propName] instanceof Array && obj[propName] instanceof Array) {
            if (!this[propName].equals(obj[propName])) {
                return false;
            }
        } else if (this[propName] instanceof Object && obj[propName] instanceof Object) {
            if (!this[propName].equals(obj[propName])) {
                return false;
            }
        } else if (this[propName] != obj[propName]) {
            return false;
        }
    }

    return true;
};
```



# 防抖和节流





# 深拷贝和浅拷贝





# 单例模式





# 数组去重



# 手写 promise.all 和 promise.race





# 模拟实现 new





# 实现 call/apply/bind



# 模拟实现 Object.create



# 千分位分隔符

数字（考虑数字是否合法，正负号、小数点）、字符串





# 实现三角形



# 实现三栏布局/双栏布局















































































