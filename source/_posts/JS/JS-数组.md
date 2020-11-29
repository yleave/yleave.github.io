---
title: JS 数组
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/js.jpg'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/65.jpg'
date: 2020-11-27 16:16:39
categories:
    - JS
tags:
    - JS Array
---


# 数组

## 获取数组中的最大、最小值

```js
let arr = [3, 5, 1, 7, 4];
console.log(Math.max(...arr)); // 7
console.log(Math.min(...arr)); // 1
```

## 创建多维数组

&emsp;&emsp;以创建 `5x5` 的**二维数组**为例：

```js
let arr = new Array(5).fill(0).map(_ => new Array(5).fill(0))；
```

&emsp;&emsp;==注意==，以下写法是**错误**的：

```js
let arr = new Array(5).fill(new Array(5).fill(0));
```

&emsp;&emsp;这种写法中，第二维的数组其实都是相同的，所以你改变一个元素如 `arr[0][1]`，那么 `arr[0][1]`、`arr[1][1]`、...、`arr[4][1]` 的元素都会被一同改变，因为这些数组的引用都是相同的。

## 数组去重

### 1.使用 set 去重

```js
const array = [1, 2, 3, 3, 5, 5, 1];
const uniqueArray = [...new Set(array)];
console.log(uniqueArray); // [1, 2, 3, 5]
```

### 2.使用 filter 去重

&emsp;&emsp;这种方法中，对于重复的数字，只有从左到右的第一个数字会满足条件，返回 `true`。

```js
let arr = [1, 2, 2, 4, 1, 3, 5, 4];

let n_arr = arr.filter((val, index, array) => {
    return array.indexOf(val) === index;
});

// [1, 2, 4, 3, 5]
```

### 3.数组遍历

&emsp;&emsp;创建一个新数组，遍历原来的数组，若当前数字不在新数组中则加入新数组，这种方法与 `filter` 的思想是相同的：

```js
let arr = [1, 2, 2, 4, 1, 3, 5, 4];
let n_arr = [];

for (let v of arr) {
    if (n_arr.indexOf(v) === -1) {
        n_arr.push(v);
    }
}

console.log(n_arr); // [1, 2, 4, 3, 5]
```

## 判断两个数组的内容是否相同

&emsp;&emsp;不能使用 `===` 来判断，因为数组也是对象，`===` 对于对象会判断其引用是否相同来判断真假。

### 1. 数组排序 + 转字符串

&emsp;&emsp;一个判断数组元素是否相同的方法是：**先将数组转为字符串，对字符串进行排序或转换前对数组进行排序，再来判断字符串是否相同**。

&emsp;&emsp;字符串转换的方法有 `arr.toString()` 和 `JSON.stringify(arr)` 与 `arr.join()` 等。

```js
let a = ['a','b','c','d']; b=['b','c','d','a'];
function isEqual(arr1 = [],arr2 = []){
    return JSON.stringify(arr1.sort()) == JSON.stringify(arr2.sort())
}
isEqual(a,b)
```

### 2. 编写 Array.prototype.equals

&emsp;&emsp;如果不仅要判断元素是否相同还要判断**元素的顺序**是否相同且数组内有**嵌套关系**的话可以使用以下方法：

```js
// Warn if overriding exiting method
if (Array.prototype.equals) {
    console.warn('Overriding existing Array.prototype.equals.');
}

Array.prototype.equals = function(arr) {
    // If the other array is a falsy value, return
    if (!arr) {
        return false;
    }

    // comapre lengths - can save a lot of time
    if (this.length != arr.length) {
        return false;
    }

    for (let i = 0, l = this.length; i < l; ++i) {
        // Check if we have nested arrays
        if (this[i] instanceof Array && arr[i] instanceof Array) {
            // Recurse into the nested arrays
            if (!this.equals(arr[i])) {
                return false;
            }
        } else if (this[i] instanceof Object && arr[i] instanceof Object) {
            // Requires Object.equals !!!
            if (!this[i].equals(arr[i])) {
                return false;
            }
        } else if (this[i] != arr[i]) {
            return false;
        }
    }

    return true;
}

// Hide method from for in loops
Object.defineProperty(Array.prototype, 'equals', {enumerable: false});
```

上面代码中，`Objec` 的 `equals` 方法需要自己实现，一个 [Object.prototype.equals](https://yleave.top/2020/11/27/JS/JS-%E5%AF%B9%E8%B1%A1/#%E6%AF%94%E8%BE%83%E5%AF%B9%E8%B1%A1%E4%B8%AD%E7%9A%84%E5%86%85%E5%AE%B9%E6%98%AF%E5%90%A6%E7%9B%B8%E5%90%8C%E7%BC%96%E5%86%99-objectequals-%E5%87%BD%E6%95%B0)

