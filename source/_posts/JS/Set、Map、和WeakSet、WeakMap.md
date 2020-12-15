---
title: Set、Map、和WeakSet、WeakMap
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/48.png'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/76.jpg'
date: 2020-12-09 20:17:41
categories:
	- JS
tags:
	- JS
	- Map
	- WeakMap
	- Set
	- WeakSet
---



转载于 [sisterAn/blog](https://github.com/sisterAn/blog/issues/24)

# 集合（Set）

`Set` 是 ES6 新增的数据结构，类似于数组，但其成员是唯一无序的，没有重复的值。

`Set` 对象可根据 `iterable` 对象来创建：

```js
let arr = [1, 2, 3, 4, 1, 2, 3];

let s = new Set(arr); // Set(4) {1, 2, 3, 4}

// 数组去重
let uniArr = [...s];	// [1, ,2, 3, 4]
```

`Set` 对象允许存储任何类型的唯一值，**无论是基本数据类型的值还是对象引用**。

```js
let s = new Set([1, 2]); // 存储值 1 和 2
let obj = {a: 1, b: 2};
let obj2 = {a: 1, b: 2};
let obj3 = obj;
s.add(obj);
s.add(obj2); // 两个对象引用不同，因此能正常存储
s.add(obj3); // 引用与 obj 相同，无法存储
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201209203523436.png" alt="image-20201209203523436" style="zoom:80%;" />

向  `Set` 中加入值时，不会发生类型转换，因此 `5` 和 `"5" ` 是两个不同的值（ `Object` 中键 的存储都会转为字符串）

Set 内部判断两个值是否不同，使用的算法叫做“Same-value-zero equality”，它类似于**精确相等**运算符（`===`），主要的区别是**Set 认为`NaN`等于自身，而精确相等运算符认为`NaN`不等于自身。**

```js
let set = new Set();
let a = NaN;
let b = NaN;
set.add(a);
set.add(b);
set // Set {NaN}

let set1 = new Set()
set1.add(5)
set1.add('5')
console.log([...set1])	// [5, "5"]
```

## Set 实例属性

### constructor

构造函数

### size

元素数量 

注意不是 `length`

```js
let set = new Set([1, 2, 3, 2, 1])

console.log(set.length)	// undefined
console.log(set.size)	// 3
```

## Set 实例方法

### 操作方法

- `add(value)`：向集合中添加新元素
- `delete(value)`：从集合中移除元素
- `has(value)`：判断集合中是否存在某个元素
- `clear()`：情况集合

### 遍历方法

遍历属性为插入顺序。

- `keys()`：返回一个包含集合中所有键的迭代器

- `values()`：返回一个包含集合中所有值的迭代器

- `entries()`：返回一个包含集合中所有元素的键值对的迭代器

- `forEach(callback, thisArg)`：对集合中的元素执行 `callback` 操作，若提供了 `thisArg` 参数，回调中的 `this` 会是这个值。

  `Set` 中没有什么键值之分，键 和 值都是 `Set` 中的元素。

  ```js
  let s = new Set(['a', 1, true, {i: 3, j: 4}]);
  
  console.log('keys: ', s.keys());
  
  console.log('values: ', s.values());
  
  console.log('entries: ', s.entries());
  ```

  <img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210140730077.png" alt="image-20201210140730077" style="zoom:80%;" />

注意到这几个方法返回的都是一个迭代器，因此可以使用 `next` 方法来遍历：

```js
let iter = s.keys();

while (true) {
    let item = iter.next();
    if (item.done) {
        break;
    }
    console.log(item);
}
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210144517054.png" alt="image-20201210144517054" style="zoom:80%;" />



还可使用 `for ··· of` 来遍历：

```js
for (let item of s.keys()) {
    console.log(item); 
}

// 返回结果相同
for (let item of s) {
    console.log(item);
}

s.forEach((value, key) =>  {
    console.log(key + ' : ' + value)
})
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210144632720.png" alt="image-20201210144632720" style="zoom:80%;" /><img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210144803339.png" alt="image-20201210144803339" style="zoom:80%;" />

`Set` 可使用 `map` 、`filter` 方法：

```js
let set = new Set([1, 2, 3])
set = new Set([...set].map(item => item * 2))
console.log([...set])	// [2, 4, 6]

set = new Set([...set].filter(item => (item >= 4)))
console.log([...set])	//[4, 6]
```

因此，Set 很容易实现**交集**（Intersect）、**并集**（Union）、**差集**（Difference）

```js
let s1 = new Set([1, 2, 3]);
let s2 = new Set([2, 3, 4]);

let union = new Set([...s1, ...s2]);
console.log(union); // Set(4) {1, 2, 3, 4}

let intersect = new Set([...s1].filter(val => s2.has(val)));
console.log(intersect); // Set(2) {2, 3}

let difference = new Set([...s1].filter(val => !s2.has(val)));
console.log(difference); // Set(1) {1}
```

# WeakSet

`WeakSet` 对象允许你将**弱引用对象**储存在一个集合中

`WeakSet` 与 `Set` 的区别：

- `WeakSet` 只能储存对象引用，不能存放值，而 `Set` 对象都行。
- `WeakSet` 对象中储存的对象值都是被弱引用的，即**垃圾回收机制不考虑 `WeakSet` 对该对象的引用**，如果没有其他变量或属性引用这个对象，那么这个对象会被回收掉。所以， `WeakSet` 对象中有多少个成员，取决于垃圾回收机制有没有进行回收，回收后成员个数可能不一样（被垃圾回收了），`WeakSet` 对象是无法被遍历的（ES6规定），也没有办法拿到它所包含的所有元素。



成员都是弱引用，可以被垃圾回收机制回收，**可以用来保存DOM节点，不容易造成内存泄漏**

**weakSet**可以用来保存DOM节点， 当节点被删除， **weakSet**里面的该节点如果不存在别的引用的话， 一段时间内会被内存回收；



为了验证弱引用机制，先输出一次 `weakset`，然后在 `10` 秒后再输出一次，可以看到，为被其他变量引用的 `[1, 2]` 就被垃圾回收了：

```js
let a = [3, 4];
let wks = new WeakSet([[1, 2], a]);

console.log(wks);

setTimeout(() => {
    console.log(wks);
}, 10000)
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210150658051.png" alt="image-20201210150658051" style="zoom:80%;" />

同时根据上图可看到 `WeakSet` 中的方法：`add(value)`、`has(value)`、`delete(value)` 和构造器方法 `constructor`

# 字典 Map

集合`set` 与字典 `map` 的区别：

- 共同点：都可以存储**不重复**的值
- 不同点：集合是以 `[value, value]` 的方式存储，而字典是以 `[key, value]` 的方式存储，因此字典可以根据 `key` 来快速的获取其对应的 `value`。

```js
let mp = new Map();
mp.set(1, 'a');
mp.set('1', 'a');
mp.set({a: 1}, 3);

console.log(mp);
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210151834885.png" alt="image-20201210151834885" style="zoom:80%;" />

如何具有 `Iterator` 接口，且每个成员都是一个双元素的数组的数据结构都可以当作 `Map` 构造函数的参数，如：

```js
let set = new Set([['foo', 1], ['bar', 2]]);
console.log(set);

let map = new Map(set);
console.log(map);

let m1 = new Map([['foo', 1], ['bar', 2]]);
let m2 = new Map(m1);
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210152321089.png" alt="image-20201210152321089" style="zoom:80%;" />



若 `map` 的键是一个对象，那么会根据对象的引用来判断是否是同一个对象，且 `NaN` 虽然不严格相等于自身，但 `map` 将其视为同一个键。

```js
let map = new Map();

map.set([1, 2], 'a');
map.set([1, 2], 'b');
map.set(NaN, 'c');
map.set(NaN, 'd');

console.log(map);
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210153059720.png" alt="image-20201210153059720" style="zoom:80%;" />

## Map 的属性

### constructor

### size

字典中所包含元素的个数

## 操作方法

- `set(key, value)` ：向字典中添加元素
- `get(key)`：通过键来查找对于的值
- `has(key)`：判断字典中是否存在键值对
- `delete(key)`：通过 `key` 从字典中移除对应键值对
- `clear()`：将字典清空

## 遍历方法

- `keys()`：将字典中的所有键以迭代器的形式返回
- `values()`：将字典中的所有值以迭代器的形式返回
- `entries()`：将字典中的所有键值对以迭代器的方式返回
- `forEach()`：遍历字典中的所有成员

- `for···of` ：遍历字典中的所有成员

```js
for (let item of map) {
    console.log(item); // item 是一个数组
    console.log('key: ', item[0], 'value: ', item[1]);
}
```

## 与其他数据结构的相互转换

### map 转 array

```js
const map = new Map([[1, 1], [2, 2], [3, 3]])
console.log([...map])	// [[1, 1], [2, 2], [3, 3]]
```

### array 转 map

```js
const map = new Map([[1, 1], [2, 2], [3, 3]])
console.log(map)	// Map {1 => 1, 2 => 2, 3 => 3}
```

### map 转 object

因为 `Object `的键名都为字符串，而`Map `的键名为对象，所以**转换的时候会把非字符串键名转换为字符串键名**。

```js
const map = new Map();
map.set('️⚽️', 'soccer');
map.set('⚾️', 'baseball');
map.set('?', 'tennis');
```

#### 方法 1

```js
const obj = Array.from(map).reduce((obj, [key, value]) => 
	Object.assign(obj, { [key] : value })
, {});
```

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210210723884.png" alt="image-20201210210723884" style="zoom:80%;" />

首先，`Array.from(map)`，会变成这样：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/image-20201210211724893.png" alt="image-20201210211724893" style="zoom:80%;" />

然后，在 `reduce` 回调函数中每次将新的键值对加入到 `{}` 中。

不过这种方式在数据量过大的时候，在每个迭代器中创建一个新对象（`Object.assign`）时，性能会受到影响。

#### 方法 2

方法 2 优化了方法 1 中的性能影响：

```js
const obj = Array.from(map).reduce((obj, [key, value]) => {
    obj[key] = value;
    return obj;
}, {});
```

使用 `Array.from(map).reduce(fn, {})`，可以安全的在累加器中操作 `object`

#### 方法 3

使用拓展运算符`...`来替换 `Array.from`：

```js
const obj = [...map.entries()].reduce((obj, [key, value]) => (obj[key] = value, obj), {})
```



**经过测试，效率上：方法 3 > 方法2 > 方法1**

### object 转 map

```js
let map = new Map();
for (let [key, value] of Object.entries(obj)) {
    map.set(key, value);
}
```

### map 转 JSON

```js
const json = JSON.stringify([...map]);

// [["️⚽️","soccer"],["⚾️","baseball"],["?","tennis"]]
console.log(json)  
```

### JSON 转 map

```js
const json = '[["️⚽️","soccer"],["⚾️","baseball"],["?","tennis"]]';
const map = new Map(JSON.parse(json)); //数组对象
const map = new Map(JSON.parse(objectToMap(json))) // 普通对象 {"a":1,"b":2}
```

# WeakMap

`WeakMap` 对象与 `Map` 对象类似，是一组键值对的集合，区别是其中的**键是弱引用对象，而值可以是任意**

与 `WeakSet` 中相同，`WeakMap` 中存在的键值对与垃圾回收机制有关，若存储在 `WeakMap` 中的键是一个对象且没有被其他变量或属性引用，那么会被垃圾回收机制回收。因此 `WeakMap` 中的 `key` 也是无法枚举的。

## 方法

- `has(key)`
- `get(key)`
- `set(key, value)`
- `delete(key)`











































