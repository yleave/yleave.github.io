---
title: 手撕内核源码
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/眼镜.jpg'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/44.png'
date: 2020-12-13 22:16:40
categories:
    - 前端面试题
tags:
    - 内核源码
---

# Array

## Array.prototype.map

### map 概念

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







































































































