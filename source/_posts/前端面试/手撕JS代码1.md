---
title: 手撕JS代码1
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/js.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/54.jpg'
hide: false
date: 2020-1-13 22:16:40
categories:
    - 前端面试题
tags:
    - JS手撕系列
---


# 1. 节流和防抖

&emsp;&emsp;对于一些频繁的操作，如对窗口的 `resize`、`scroll`、输入框内容改动响应时，如果相应处理函数没有频率限制的话，会加重浏览器的负担，导致用户体验差，而防抖(debounce) 和节流(throttle) 可以有效减少处理函数的调用频率，同时不影响实际效果。

## 2.1 防抖

&emsp;&emsp;触发高频事件后 `n` 秒内函数只会执行一次，如果 `n` 秒内高频事件再次被触发，则重新计算时间。

&emsp;&emsp;一个连续操作中的处理，只触发一次，从而实现防抖动。

> 思路：使用一个定时器延迟调用处理函数，每次触发事件后都取消之前延迟调用的方法

```js
function debounce(fn, wait) {
    let timeout = null;
    return function() {
        clearTimeout(timeout); // 每当用户输入的时候把前一个 setTimeout clear 掉
        // 然后又创建一个新的 setTimeout, 这样就能保证输入字符后的 interval 间隔内如果还有字符输入的话，就不会执行 fn 函数
        timeout = setTimeout(() => {
            // this 确保当前指向的对象是调用函数的对象，如 input 对象
            fn.apply(this, arguments);
        }, wait);
    };
}
```

### 1. 立即执行的防抖

&emsp;&emsp;`underscore` 中的防抖还可以实现立即执行的功能：当 `immediate` 为 `true` 时，立即执行函数， `wait` 秒后才能重新触发（即当 `timer` 为 `null` 时）。

```js
function immedDebounce(fn, wait, immediate) {
    let timer = null;
    return function() {
        if (timer) {
            clearTimeout(timer);
        }

        if (immediate) {
            let callNow = !timer;

            timer = setTimeout(() => {
                timer = null;
            }, wait);

            if (callNow) {
                fn.apply(this, arguments);
            }
        } else {
            timer = setTimeout(() => {
                fn.apply(this, arguments);
            }, wait);
        }
    }
}
```

### 2 带返回值的 debounce

&emsp;&emsp;对于立即执行的 `debounce`，当 `immediate` 为 `true` 时，当函数立即执行时，可以直接返回函数结果，而当 `immediate` 为 `false` 时，因为设置了 `timeout` ，因此获取不到返回值，函数返回 `undefined`。

```js
function retDebounce(fn, wait, immediate) {
    let timer = null;
    return function() {
        let result;
        if (timer) {
            clearTimeout(timer);
        }

        if (immediate) {
            let callNow = !timer;

            timer = setTimeout(() => {
                timer = null;
            }, wait);

            if (callNow) {
                result = fn.apply(this, arguments);
            }
        } else {
            timer = setTimeout(() => {
                fn.apply(this, arguments);
            }, wait);
        }

        return result;
    }
}
```

### 3. 可取消的 debounce

&emsp;&emsp;若防抖时间较长，而 `immediate` 为 `true` 时（立即执行一次，然后需要再等待 `wait` 后才能触发），又想早点再次触发，可以选择取消防抖状态。

```js
function cancelDebounce(fn, wait, immediate) {
    let timer = null;
    const debounced = function() {
        let result;

        if (timer) {
            clearTimeout(timer);
        }

        if (immediate) {
            let callNow = !timer;

            timer = setTimeout(() => {
                timer = null;
            }, wait);

            if (callNow) {
                result = fn.apply(this, arguments);
            }
        } else {
            timer = setTimeout(() => {
                fn.apply(this, arguments);
            }, wait);
        }

        return result;
    };

    debounced.cancel = function() {
        clearTimeout(timer);
        timer = null;
    };

    return debounced;
}
```

### 4. 防抖实例

&emsp;&emsp;使用实例：两个按钮，当点击了 `say hello` 按钮后，需要等待两秒后再次点击按钮才能输出 `hello`，若不想等待，点击 `cancel hello` 后再次点击 `say hello` 后会立即输出 `hello`。

```js
function hi() {
    console.log('hello');
}

let inp = document.createElement('button');
inp.innerHTML = 'say hello';

let inp1 = document.createElement('button');
inp1.innerHTML = 'cancel hello';

document.body.appendChild(inp);
document.body.appendChild(inp1);

let de = cancelDebounce(hi, 2000, true);
inp.addEventListener('click', de);
inp1.addEventListener('click', de.cancel);
```





## 2.2 节流

&emsp;&emsp;高频事件触发，但在 `n` 秒内只会执行一次，所以节流会稀释函数的执行频率。



&emsp;&emsp;一个连续操作中的处理，按照阀值时间间隔进行触发，从而实现节流。

&emsp;&emsp;在一定的时间间隔内，某个事件只会执行一次

> 思路：每次触发事件时都判断当前是否有在等待执行的回调函数

### 1. 节流 - 使用定时器

&emsp;&emsp;使用了定时器，当**事件触发时不会立即执行**，等待时间过后才会开始执行，然后清空计数器。

```js
function throttle1(fn, wait) {
    let timer = null;
    return function() {
        if (!timer) {
            timer = setTimeout(() => {
                timer = null;
                fn.apply(this, arguments);
            }, wait);
        }
    }
}
```

### 2. 节流 - 使用时间戳

&emsp;&emsp;根据时间戳来判断两次响应的间隔是否大于设置的间隔，大于才能执行下一次响应函数。这种方法**响应函数 `fn` 会立即执行。**

```js
function throttle(fn, time) {
    let activeTime = 0;
    reutrn () => {
        const current = Date.now(); // +new Date()
        if (current - activeTime > time) {
            fn.apply(this, arguments);
            activeTime = current;
        }
    };
}
```



&emsp;&emsp;其实使用定时器也可以达到**立即执行**的效果：

```js
function throttle(fn, wait) {
    let timer = null;

    return function() {
        if (!timer) {
            // 将执行函数放外面 就有立即执行的效果了
            fn.apply(this, arguments);
            timer = setTimeout(() => {
                timer = null;
            }, wait);
        }
    };
}
```

### 3. 节流 - 综合写法 ： 立即执行效果 + 尾执行

&emsp;&emsp;比较上面两个节流方法：

1. 使用**时间戳**的节流方法触发事件时会**立即执行**，而使用**定时器**的方法会在触发事件 **`n` 秒后再执行**；
2. 使用**时间戳**的方法**停止触发事件后不会再执行**，使用**定时器**的方法会在**停止触发事件的 `n` 秒后再执行一次。**



&emsp;&emsp;可以综合时间戳和定时器的效果，当触发事件时会立即执行，且在停止事件触发的 `n` 秒后还会再执行一次。

```js
function throttle2(fn, wait) {
    let previous = 0, timer = null;

    let later = function() {
        timer = null;
        previous = +new Date();
        fn.apply(this, arguments);
    };

    let throttled = function() {
        let now = +new Date();
        // 剩余等待时间
        let remain = wait - (now - previous);
        // 若没有剩余时间或因修改了系统时间导致剩余时间大于等待时间
        // 若非主动修改系统时间，此时会是第一次触发执行
        if (remain <= 0 || remain > wait) {
            // 若前面有在等待时间内设置的 timer，需要将其清除，否则可能造成连续调用的情况
            if (timer) {
                clearTimeout(timer);
                timer = null;
            }

            previous = now;
            fn.apply(this, arguments);
        } else if (!timer) {
            // 否则若处于等待时间内触发且前面的定时器被清除，那么会延迟执行，延迟时间即为剩余等待时间
            timer = setTimeout(later, remain);
        }
    };

    return throttled;
}
```

### 4. 根据参数调整节流效果

&emsp;&emsp;有时候我们希望节流效果为 “有头无尾”， 有时候希望是 “有尾无头”。

&emsp;&emsp;这样的话我们可以通过设置参数 `options` 来调整，约定：

- `leading: false` ：表示禁用第一次执行
- `trailing: false` ：表示禁用停止触发的回调

```js
// 当设置了 leading 为 false 时，表示禁用第一次执行，当 trailing 为 false，表示禁用停止后的回调
function throttle3(fn, wait, options={}) {
    let timer = null, previous = 0;

    let later = function() {
        timer = null;
        // leading 为 false 时，将 previous 设置为 0，那么每次触发时，remain 计算结果都会是 wait
        previous = options.leading === false ? 0 : new Date().getTime();
        fn.apply(this, arguments);
    };

    let throttled = function() {
        let now = +new Date();
        // 当 leading 为 false 时，每次执行 fn，previous 都会被设置为 0，再加上初始时刻 previous = 0
        // 就达到了取消第一次立即执行的效果，也就是会跳过接下来的第一个 if 判断，直接进行 timer 设置
        if (options.leading === false && !previous) {
            previous = now;
        }
        let remain = wait - (now - previous);

        // 否则，若设置 trailing 为 false ，第一次 remain 会小于 0，从而使 fn 立即执行
        // 之后，在间隔时间之内(remain > 0)，由于 trailing 为 false，因此下面延迟设置的判断进不去
        // 从而达到每次只有当 remain 为 0 时，才会执行 fn，且在停止触发后，不会再延迟执行了
        if (remain <= 0 || remain > wait) {
            if (timer) {
                clearTimeout(timer);
                timer = null;
            }
            previous = now;
            fn.apply(this, arguments);
        } else if (!timer && options.trailing !== false) {
            timer = setTimeout(later, remain);
        }
    };

    return throttled;
}
```

#### 设置取消功能

&emsp;&emsp;与防抖中相似，当节流中需要等待很久才能进行下一次响应时，若想提前响应，可以使用 `cancel` 来取消：

```js
throttled.cancel = function() {
    clearTimeout(timeout);
    previous = 0;
    timer = null;
};
```

#### 注意事项

&emsp;&emsp;在上面的带有 `options` 参数的实现中，`leading：false` 和 `trailing: false` 不能同时设置。

&emsp;&emsp;若同时设置的话，当将鼠标移出时，因为 `trailing` 设置为 `false` ，停止触发的时候不会设置定时器，所以只要再过了设置的时间，再移入的话，就会立即执行了，这就违反了 `leading: false` 的规则。因此，这个 `throttle` 只有三种使用方法：

```js
// 1. 开始立即执行且停止触发后过一段时间会有一个回调
continer.onmousemove = throttle(getUserAction, 1000);

// 2. 取消开始时的立即执行
continer.onmousemove = throttle(getUserAction, 1000, {
    leading: false
});

// 3. 取消停止时设置的回调
continer.onmousemove = throttle(getUserAction, 1000, {
    trailing: false
});
```



### 5. 实例

&emsp;&emsp;示例：当窗口进行缩放时，一秒内只会执行一次打印窗口大小的函数。

```js
function sayHi(e) {
    console.log(e.target.innerWidth, e.target.innerHeight);
}

window.addEventListener('resize', throttle(sayHi));
```



## 2.3 节流和防抖场景

- a、`scroll`事件：当页面发生滚动时，`scroll`事件会被频繁的触发，`1s`触发可高达上百次。在`scroll`事件中，如果有复杂的操作（特别是影响布局的操作），将会大大影响性能，甚至导致浏览器崩溃。所以，对其进行防抖、限频很重要。
- b、`click`事件：用户进行`click`事件时，有可能连续触发点击（用户本意并非双击）。该操作有可能是不小心多次连续点击，也可能是页面状况不好的情况下，期待尽快得到反馈的有意行为；但这样的操作，反而会加剧性能问题，因此也有必要考虑防抖、限频。
- c、`input`事件：如`sug`等需要通过`ajax`及时获得数据的情况，需要进行限频，防止频繁的请求发生，减少服务器压力的同时，提高页面响应性能。
- d、`touchmove`事件：同`scroll`事件类似。

# 2. 深拷贝和浅拷贝

&emsp;&emsp;浅拷贝和深拷贝都只针对于引用数据类型

&emsp;&emsp;浅拷贝和深拷贝的区别用一张图来说就是：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/13263206-c651dc07788bf561.png" style="zoom: 33%;" />

&emsp;&emsp;图中的每个节点相当于对象的层次，浅拷贝只会复制对象的第一层元素，若其有嵌套的对象，那么这些嵌套对象的引用都是相同的，而深拷贝的复制会包括这些嵌套的对象，因此复制出的对象会是完全不同的两个对象。





## 2.1 浅拷贝

### 1. Object.assign

```js
const obj2 = Object.assign({}, obj);
```

### 2. 展开运算符

```js
const obj2 = {...obj};
```

### 3. Object.create

MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create)

> **`Object.create()`** 方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。

```js
const obj2 = Object.create(obj);
```

### 4. 循环

&emsp;&emsp;在下面的循环中，因为 `Object.entries` 或 `Object.keys` 不会遍历到原型链上的属性，因此不需要使用 `obj.hasOwnProperty` 来验证是否是自身的属性，若是 `for in` 则需要。

```js
function shallowCopy(obj) {
    if (!obj || typeof obj !== 'object') {
        return obj;
    }
    const obj2 = Array.isArray(obj) ? [] : {};
    for (let [key, value] of Object.entries(obj)) {
        obj2[key] = value;
    }

    return obj2;
}
const obj2 = shallowCopy(obj);
```

### 5. slice() 、 concat()、Array.from() 方法 （ArrayObject）

&emsp;&emsp;这几个方法都不会改变原数组，它们返回一个新数组：

```js
const arr2 = arr.slice();

const arr3 = arr.concat([]); // [].concat(arr);
```

## 2.2 深拷贝

&emsp;&emsp;在进行深拷贝时需要考虑**循环引用**的问题。

&emsp;&emsp;如果遇到复杂对象，我们可以使用工具库，比如 lodash 的 [cloneDeep](https://www.lodashjs.com/docs/lodash.cloneDeep/) 方法

### 1. JSON.parse 和 JSON.stringfy

&emsp;&emsp;**该方法不能解决循环引用问题**

```js
const obj2 = JSON.parse(JSON.stringify(obj));
```

&emsp;&emsp;**缺陷**：它会抛弃对象的`constructor`，深拷贝之后，不管这个对象原来的构造函数是什么，在深拷贝之后都会变成`Object`；会忽略`undefined、任意的函数、symbol 值`；

&emsp;&emsp;这种方法能正确处理的对象只有 `Number`, `String`, `Boolean`, `Array`, 扁平对象，也就是说，只有可以转成 JSON 格式的对象才可以这样用，像`function`没办法转成 JSON；

&emsp;&emsp;因此特殊的对象无拷贝，比如 `函数、RegExp、Date、Set、Map` 等。

### 2. 循环

&emsp;&emsp;可处理 `RegExp` 、`Date`、`Array` 和 `Object` 类型的对象，考虑了原型链，且能解决循环引用问题。

&emsp;&emsp;函数一般不需要进行深拷贝。

&emsp;&emsp;避免循环引用的方法就是使用一个对象保存每次递归时的父对象，若有一个对象引用了其任意父级对象，那么就返回那个父级对象的复制即可。

&emsp;&emsp;在下面的代码中， `parent`  是一个对象，保存了三个属性：

- `originParent` ： 当前递归函数中的 `obj`，根据 `_parent.originParent` 就能找到其所有父级对象；

- `currentParent`：当前要返回的克隆对象，因为对象是保存其引用，因此若当前对象下面有属性与其父级对象循环引用，那么该属性值就是那个父级对象的克隆对象。

  比如这个例子中的:

  ```js
  const obj1 = {
      f: 'other obj',
      obj: {
          innerObj: obj
      }
  };
  
  obj.j = obj1;	// obj1.obj 引用了父级对象 obj
  obj.b = obj;	// 循环引用自身
  ```

  那么在递归函数中就会判断到 `obj1.obj` 造成了循环引用，因此其属性值就是父级对象 `obj` 的拷贝结果。

- `parent`：用于得到每一级的父级对象

```js
function deepCopy(obj, parent = null) {
    // 先排除空值和基本数据类型
    if (!obj || typeof obj !== 'object') {
        return obj;
    }

    let o, _parent = parent;

    // 处理循环引用问题
    while (_parent) {
        if (_parent.originParent === obj) {
            return _parent.currentParent;
        }

        _parent = _parent.parent;
    }

    if (obj instanceof RegExp) {    // 处理正则
        o = new RegExp(obj.source, obj.flags);
    } else if (obj instanceof Date) {   // 处理日期类型
        o = new Date(obj.getTime());
    } else {    // 数组和对象统一处理，因为可能需要递归复制
        if (obj instanceof Array) { 
            o = new Array();
        } else {    // 若是对象，继承其原型
            let proto = Object.getPrototypeOf(obj);
            o = Object.create(proto);
        }
        for (let [key, val] of Object.entries(obj)) {
            if (val && typeof val === 'object') {
                val = deepCopy(val, {
                    originParent: obj,
                    currentParent: o,
                    parent: parent
                });
            }
            o[key] = val;
        }
    }

    return o;
}
```

### 3. 更完备的深拷贝

&emsp;&emsp;使用了 `WeakMap` 解决了循环引用问题，且不会造成内存泄漏

&emsp;&emsp;能应对 `RegExp`、`Date`、`Function`、`Map`、`Set` 等特殊对象的拷贝

```js
function deepClone(obj, map=new WeakMap()) {
    // 处理 null 和 undefined
    if (obj == null) return obj;

    // 若是基本类型，直接返回
    if (typeof obj !== 'object' && typeof obj !== 'function') return obj;

    // 处理 Date 和 RegExp
    if (obj instanceof Date) return new Date(obj);
    if (obj instanceof RegExp) return new RegExp(obj.source, obj.flags);

    // 使用 map 解决循环引用问题
    if (map.has(obj)) return map.get(obj);

    // 处理函数对象 返回一个新函数，在调用这个函数时会返回原本函数的执行结果
    if (obj instanceof Function) {
        return function() {
            return obj.apply(this, [...arguments]);
        }
    }

    // 下面是 数组/普通对象/Set/Map 的处理

    // 从其原型链中继承的 constructor
    const res = new obj.constructor();

    // 设置 map 以处理循环引用问题
    map.set(obj, res);

    if (obj instanceof Map) {
        obj.forEach((item, index) => {
            // key(index) 不一定是基本数据类型
            res.set(deepClone(index, map), deepClone(item, map));
        });
    } else if (obj instanceof Set) {
        obj.forEach((item) => {
            obj.add(deepClone(item, map));
        });
    } else {
        for (let [key, value] of Object.entries(obj)) {
            if (value && typeof value === 'object') {
                res[key] = deepClone(value, map);
            } else {
                res[key] = value;
            }
        }
    }

    return res;
}
```





# 3. JS  

## 3.1 instanceof

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof)

&emsp;&emsp;**`instanceof`** **运算符** 用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。

&emsp;&emsp;故：`instanceof `操作符其实就是检查左侧的元素的 **__proto__**链上有没有右侧类或对象的`prototype`存在。

> 思路：顺着原型链逐层查找，直到原型链的尽头 `null` 为止，若过程中 `left` 的原型与 `right` 的原型相同，则返回 `true`。

```js
function myInstanceof(left, right) {
    // 首先，对于基本数据类型，一律返回 false
    if (!left|| typeof left !== 'object') {
        return false;
    }

    // 获取左边的原型
    let proto = Object.getPrototypeOf(left);

    while (true) {
        if (proto === null) return false;
        if (proto === right.prototype) return true;
        proto = Object.getPrototypeOf(proto);
    }
}
```



## 3.2 Object.create

[MDN polyfill](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/create#polyfill)

> 创建一个纯净的新对象，然后继承其原型

```js
Object.prototype.myCreate = function(proto) {
    if (!proto || (typeof proto !== 'object' && typeof proto !== 'function')) {
        throw new TypeError('proto must be an object!' + proto);
    }
    function F() {}
    F.prototype = proto;

    return new F();
}
```



## 3.3 new

[MDN new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new)

`new` 被调用后做了三件事：

- 让实例对象可以访问到构造函数的属性
- 让实例对象可以访问构造函数原型所在原型链上的属性
- 考虑构造函数有返回值且返回值为对象的情况，这时候返回的对象的优先级更高


```js
function myNew(ctor, ...args) {
    let fn = Array.prototype.shift.call(arguments);
    if (typeof fn !== 'function') throw `${fn} is not a constructor`;
    let obj = Object.create(fn.prototype);	// 创建一个新的对象，且继承其原型
    let res = fn.apply(obj, args);
    let isObject = res && typeof res === 'object';
    let isFunction = typeof res === 'function';
    return isObject || isFunction ? res : obj;
}
```



## 3.4 call & apply

[MDN call](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

[MDN apply](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

> call()方法的作用和 apply() 方法类似，区别就是`call()`方法接受的是**参数列表**，而`apply()`方法接受的是**一个参数数组**。

&emsp;&emsp;主要实现思路就是将需要被调用的函数作为需要使用的 `this` 指向的对象的属性值，然后调用即可，它会自动绑定 `this`。

```js
Function.prototype.myCall = function(obj, ...args) {    // or myApply
    if (typeof this !== 'function') throw 'caller must be a function';
    let self = arguments[0] || window;  // 获取 this 指向的对象，也就是 参数 obj
    self._fn = this;
    let _args = [...arguments].flat().slice(1); // 获取展开后的参数列表，且去除第一个参数 this 指向，使用 flat 展开的原因：若是 apply 函数，传入的参数会是一个数组，因此需要展开
    let res = self._fn(..._args);   // 因为是 self 调用的 fn, 因此 fn 中的 this 是指向 self 的
    Reflect.deleteProperty(self, '_fn');  // 将 _fn 从 self 中删除
    return
    res;
}
```









## 3.5 bind

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

> `bind()` 方法创建一个新的函数，在 `bind()` 被调用时，这个新函数的 `this` 被指定为 `bind()` 的第一个参数，而其余参数将作为新函数的参数，供调用时使用。

```js
Function.prototype.myBind = function() {
    if (typeof this !== 'function') throw 'caller must be a function!';
    let slice = Array.prototype.slice;
    let self = this;
    let context = arguments[0];
    let args = slice.call(arguments, 1);

    return function() {
        // 调用 bind 时会返回一个新函数，然后新函数后还会传递参数，拼接这些参数
        let funcArgs = args.concat(slice.call(arguments));
        // 最后返回函数调用结果
        return self.apply(context, funcArgs);
    };
};
```



此外，`bind` 还有一个特点：

> 一个绑定函数也能使用`new`操作符创建对象：这种行为就像把原函数当成构造器。提供的 `this `值被忽略，同时调用时的参数被提供给模拟函数。

也就是说，当 `bind` 返回的函数作为构造器时，绑定的 `this` 值将失效，而传入的参数依然生效.



```js
Function.prototype.myBind = function() {
    if (typeof this !== 'function') throw new TypeError('caller is not a function!');

    let slice = Array.prototype.slice;
    let self = this;
    let context = arguments[0];
    let args = slice.call(arguments, 1);

    let fBound = function() {
        let newArgs = args.concat(slice(arguments));

        // 当 fBound 作为构造函数时，this 指向实例，因此 instanceof 返回 true，因此传入 this 让实例继承来自绑定函数的值
        // 当 fBound 作为普通函数时，this 指向 wondow，因此会将 this 绑定到 context
        return self.apply(this instanceof fBound ? this : context, newArgs);
    };

    // 修改返回函数的 prototype 为绑定函数的 prototype ，实例就可以继承绑定函数原型中的值
    fBound.prototype = Object.create(self.prototype);

    return fBound;
};
```







## 3.6 Array.map

**map 概念：**

&emsp;&emsp;`map(callback(val, idx, arr), thisArg)` 方法将**创建一个新数组**，这个数组中的元素是原数组中的每个元素都调用 `callback` 后的结果，其中 `callback` 的三个参数分别是原数组中的**元素**、**元素对应索引值**和**原数组**，`thisArg` 是 `map` 函数的 `this` 指向。

&emsp;&emsp;因此调用 `map` 函数后，**原数组不会发生改变**。

&emsp;&emsp;且，调用的数组 `arr` 中的元素不一定是连续的（有的索引位置会为 `empty`），这点需要注意。

```js
Array.prototype.myMap = function(callbackFn, thisArg) {
    // null 或 undefined
    if (this == null) {
        throw new TypeError(`can't not read proterty 'map' of ${this}` );
    }

    if (Object.prototype.toString.call(callbackFn) !== '[object Function]') {
        throw new TypeError(`${callbackFn} is not a function!`);
    }

    let O = Object(this);   // 规定 this 需要先转换为对象
    let len = O.length >>> 0;   // 保证 len 为数字且为整数
    let T = thisArg || null;

    let res = new Array(len);

    for (let i = 0; i < len; ++i) {
        if (i in O) {
            let mappedValue = callbackFn.call(T, O[i], i, O);
            res[i] = mappedValue;
        }
    }

    return res;
};
```

其中：

- `>>> `运算符为 零填充右移运算符，如 `0101 >>> 1 : 0010`，保证 `len ` 为数字且为整数
- 使用 [Object](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)  是为了保证 `o` 一定是一个对象：

  - 当给定值是 `null` 或 `undefined` 时，会创建并返回一个空对象
  - 若传进去的是一个基本类型的值，则会构造其包装类型的对象，如 `Object(3)` ，会返回 `Number {3}`
  - 若传的是引用类型的值，仍会返回这个值，因此引用是相同的









# 4.  柯里化

&emsp;&emsp;柯里化，是函数式编程的一个重要概念。

&emsp;&emsp;概念：是一种将使用多个参数的函数转换成**一系列**使用一个参数的函数的技术。为了保证柯里化能达到预期结果，这个函数需要是纯函数。

&emsp;&emsp;好处：减少代码冗余，增加可读性

&emsp;&emsp;坏处：若参数过多难免会对性能造成一定的影响

> 纯函数：一个函数的返回结果只依赖于它的参数，并且在执行过程中不会产生副作用，那么这种函数就叫做纯函数

&emsp;&emsp;一个最常见的柯里化例子：

```js
function sum(a, b, c) {
    return a + b + c;
}

// 柯里化后
let curryingSum = curry(sum);

curryingSum(1, 2, 3); // => 6
curryingSum(1)(2)(3); // => 6
curryingSum(1, 2)(3); // => 6
curryingSum(1)(2, 3); // => 6
```



==新知识==：`函数名.length = 函数参数个数`

```js
function curry(fn, args=[]) {
    return function() {
        let newArgs = args.concat(Array.prototype.slice.call(arguments));
        // 若 fn 参数个数大于 当前参数列表的参数个数，那么递归调用 curry
        // 每次调用 curry 都会返回一个函数，然后这个函数中会保存当前已传入的参数
        // 一直调用 curry 直到参数个数足够，就调用 fn 返回结果
        if (fn.length > newArgs.length) {
            return curry.call(this, fn, newArgs);
        } else {
            return fn.apply(this, newArgs);
        }
    };
}

// ES6 装逼写法
const curry = fn => 
    judge = (...args) => 
        args.length === fn.length
        	? fn(...args)
        	: (arg) => judge(...args, arg)
```



参考：

https://www.jianshu.com/p/2975c25e4d71

https://segmentfault.com/a/1190000018180159

# 5. 组合寄生继承

[Object.setPrototypeOf](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)

## 组合继承

```js
function Parent() {
    this.id = '001';
}

function Child() {
    Parent.call(this);	// 继承 Parent 的属性
    this.age = 18;
}

Child.prototype = new Parent();	// 继承原型属性和方法
Child.prototype.constructor = Child;
```

&emsp;&emsp;上面这种继承方式就叫做**组合继承**，这种方法最大的缺点就是调用了两次父类的构造函数，带来的影响就是子类内部和子类的原型链上会有属性冗余，如上例中 `id` 属性就会同时存在与 `Child` 和 `Child.prototype` 中。

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210222165839847.png" alt="image-20210222165839847" style="zoom:80%;" />

## 寄生继承

&emsp;&emsp;**寄生继承** 就是借用一个中间对象来继承原对象的属性和方法，然后在这个新对象上进行二次封装，如：

```js
function object(o) {
    function F() {};
    F.prototype = o;
    return new F();
}
var twoD = {
    name: '2D shape',
    dimensions: 2
}
function triangle(s, h) {
    var that = object(twoD);
    that.name = 'Triangle';
    return that;
}
```

&emsp;&emsp;上面的 `object` 方法可以使用 `Object.create` 方法来替代。





&emsp;&emsp;**寄生组合继承** 就是结合了以上两种继承方法，用来解决组合继承中冗余的属性问题：

- 通过借用构造函数来继承属性；

- 通过原型链来继承方法。

```js
function Parent() {
    this.id = '001';
}

// 定义静态属性和方法
Parent.log = function() {
    console.log('static');
}
Parent.x = 3;

function Child() {
    Parent.call(this);  // 继承父类属性
    this.age = 18;
}

Child.prototype = Object.create(Parent.prototype);  // 继承父类原型的属性和方法
Object.setPrototypeOf(Child, Parent);   // 继承静态属性
Child.prototype.constructor = Child;    // constructor 重新指向 Child
```

&emsp;&emsp;使用寄生组合继承的话，就能解决组合继承的问题：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210222170020436.png" alt="image-20210222170020436" style="zoom:80%;" />

# 6. class 私有属性

&emsp;&emsp;私有属性一般满足：

- 能被 class 内部的不同方法访问，但是不能在类外部被访问
- 不能被子类继承

&emsp;&emsp;ES6 中已经有了一个 `#` 来创建私有属性，不过需要 `babel` 转译。

```js
const MyClass = (function() {	// 利用闭包和 WeakMap
    const _x = new WeakMap();
    class InnerClass {
        constructor(x) {
            _x.set(this, x);
        }

        getX() {
            return _x.get(this);
        }
    }

    return InnerClass;
})();

let c = new MyClass(5);
console.log(c.getX());  // 5
console.log(c._x)   // undefined
```

# 7.  数组排序

[菜鸟教程](https://www.runoob.com/w3cnote/ten-sorting-algorithm.html)

排序算法分类：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/849589-20190306165258970-1789860540.png" style="zoom:67%;" />

![](https://gitee.com/ylea/imagehost/raw/master/img/sort.png)

## 7.1 冒泡排序

&emsp;&emsp;冒泡排序的思想就是使用双循环，将相邻数字中大的数字往前冒泡。

&emsp;&emsp;若一趟排序中没有进行交换，则表示数组已经有序，可以退出了。

```js
function bubbleSort(arr) {
    let len = arr.length;
    for (let i = 0; i < len; ++i) {
        let flag = true;
        for (let j = 1; j < len - i; ++j) {
            if (arr[j] < arr[j-1]) {
                [arr[j], arr[j-1]] = [arr[j-1], arr[j]];
                flag = false;
            }
        }
        if (flag) {
            break;
        }
    }
}
```

## 7.2 简单插入排序

&emsp;&emsp;思想：分为已排序部分和未排序部分，遍历未排序部分，每次将第一个数在已排序部分中选择一个合适的位置插入。 

```js
function insertSort(arr) {
    let len = arr.length;

    for (let i = 1; i < len; ++i) {
        let val = arr[i];
        let j = i - 1;
        while (j >= 0 && arr[j] > val) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = val;
    }
}
```

## 7.3 简单选择排序

&emsp;&emsp;基本思想是每次从无序列表中选择一个最大/小的数放到有序列表的前/后面。

```js
function selectSort(arr) {
    let len = arr.length;

    for (let i = 0; i < len - 1; ++i) {
        let idx = i;
        for (let j = i + 1; j < len; ++j) {
            if (arr[idx] > arr[j]) {
                idx = j;
            }

            if (idx !== i) {
                [arr[idx], arr[i]] = [arr[i], arr[idx]];
            }
        }
    }
}
```

## 7.4 快速排序

&emsp;&emsp;快排是不稳定的原因是：在排序过程中，若选择一个 `pivot`，此时将小于等于 `pivot` 的都放左边，大于的都放右边，那么就是不稳定的了；或者是将大于等于的放右边，小于等于的放左边，同样也是不稳定的。（因为除了小于后就是大于等于了）

```js
function qiuckSort(arr, left=0, right=arr.length-1) {
    if (left < right) {
        let idx = partition(arr, left, right);
        qiuckSort(arr, left, idx - 1);
        qiuckSort(arr, idx + 1, right);
    }
}
```

&emsp;&emsp;其中 `partition` 有两种写法：

```js
function partition(arr, left, right) {
    // 以 left 值为基准
    let index = left + 1;
    let pivot = arr[left];
    for (let i = index; i <= right; ++i) {
        if (arr[i] < pivot) {
            [arr[i], arr[index]] = [arr[index], arr[i]];
            index++;
        }
    }
    [arr[left], arr[index-1]] = [arr[index-1], arr[left]];

    return index - 1;

    // 以 right 为基准
    // let index = left;
    // for (let i = index; i < right; ++i) {
    //     if (arr[i] < arr[right]) {
    //         [arr[i], arr[index]] = [arr[index], arr[i]];
    //         index++;
    //     }
    // }
    // [arr[right], arr[index]] = [arr[index], arr[right]];

    // return index;
}
```

```js
function partition(arr, left, right) {
    let pivot = arr[left];
    while (left < right) {
        while (left < right && arr[right] > pivot) {
            right--;
        }
        arr[left] = arr[right];
        while (left < right && arr[left] <= pivot) {
            left++;
        }
        arr[right] = arr[left];
    }
    arr[left] = pivot;

    return left;
}
```

## 7.5 归并排序



```js
function mergeSort(arr, left=0, right=arr.length-1) {
    if (left < right) {
        let mid = left + Math.floor((right - left) / 2);
        mergeSort(arr, left, mid);
        mergeSort(arr, mid + 1, right);
        merge(arr, left, mid, mid + 1, right);
    }
}

function merge(arr, l1, r1, l2, r2) {
    let res = [];
    let l = l1;
    while (l1 <= r1 && l2 <= r2) {
        if (arr[l1] < arr[l2]) {
            res.push(arr[l1]);
            l1++;
        } else {
            res.push(arr[l2]);
            l2++;
        }
    }

    while (l1 <= r1) {
        res.push(arr[l1]);
        l1++;
    }
    while (l2 <= r2) {
        res.push(arr[l2]);
        l2++;
    }

    for (let i = l; i <= r2; i++) {
        arr[i] = res[i - l];
    }
    
    // or arr.splice(l, r2 - l + 1, ...res);
}
```

## 7.6 堆排序

https://github.com/sisterAn/JavaScript-Algorithms/issues/60

&emsp;&emsp;堆主要是使用数组来存储一棵树，假设有一个数组的第一个有意义的元素索引从 `1` 开始，对于一个索引 `i`，那么其父节点的索引就是 `i / 2`，其左孩子索引是 `i * 2`，右孩子索引值是 `i * 2 + 1`。

&emsp;&emsp;若堆索引从 `0` 开始，那么对于一个索引 `i`，其父节点索引就是 `(i-1)/2`，左孩子索引就是 `i * 2 - 1`，右孩子索引就是 `i * 2`。



&emsp;&emsp;**下面的算法是按照堆元素索引从 `1` 开始的。**

&emsp;&emsp;因此，首先需要有一个长度为 `1` 的数组存储堆元素：

```js
let items = [,];
```

 

&emsp;&emsp;首先，给你一个数组，需要将其调整为堆。

&emsp;&emsp;堆的调整有两种方法：

- **自下而上堆化**：将节点值与父节点值比较，若大于父节点值（大顶堆）或小于父节点值（小顶堆），那么就将其交换
- **自上而下堆化**：对于一个节点，比较其左右孩子的节点值，选出其较大的子节点值（大顶堆）和当前节点值比较，若子节点值大于当前节点值，则交换位置；小顶堆同理

&emsp;&emsp;**所以，自下而上式堆是调整节点与父节点（往上走），自上往下式堆化是调整节点与其左右子节点（往下走）。**



### 从前往后，自下而上堆化

```js
function buildHeap(arr, heapSize) {
    while (heapSize < arr.length - 1) {
        heapSize++;
        heapify(arr, heapSize);
    }
}

// 大顶堆
function heapify(arr, i) {
    while (Math.floor(i / 2) > 0 && arr[i] > arr[Math.floor(i / 2)]) {
        swap(arr, i, Math.floor(i / 2));
        i = Math.floor(i / 2);
    }
}

function swap(arr, l, r) {
    [arr[l], arr[r]] = [arr[r], arr[l]];
}

let items = [, 3, 5, 1, 2, 4];

buildHeap(items, 1);

console.log(items)  // 5 4 1 2 3
```



### 从后往前，自上而下堆化

&emsp;&emsp;这边说的 **后** 不是数组末尾，而是树中的最后一个非叶子结点，索引值就是 `heapSize / 2`。

```js
function buildHeap(heap, heapSize) {
    for (let i = Math.floor(heapSize / 2); i > 0; --i) {
        heapify(heap, heapSize, i);
    }
}

// 大顶堆
function heapify(heap, heapSize, i) {
    while (true) {
        // 使用 maxIndex 记录左右孩子结点的较大结点值索引，当 maxIndex === i 或 当前节点无子节点时，退出循环
        let maxIndex = i;
        if (i * 2 <= heapSize && heap[i * 2] > heap[i]) {
            maxIndex = i * 2;
        }
        if (i * 2 + 1 <= heapSize && heap[i * 2 + 1] > heap[maxIndex]) {
            maxIndex = i * 2 + 1;
        }

        if (maxIndex === i) break;
        swap(heap, i, maxIndex);
        i = maxIndex;
    }
}
```



### 堆排

&emsp;&emsp;堆其实就是一棵**完全二叉树**，因此可以使用数组存储，根据索引值来计算其父节点和子节点的位置。

&emsp;&emsp;设根结点的索引值为 `1`，那么我们若要对其进行排序，我们先创建一个堆，堆顶元素即是最大/最小值。然后我们每次将堆顶元素与堆的最后一个节点交换位置，并将最后一个节点从堆中移除（堆的大小减 1），移除堆尾元素的堆仍是一个完全二叉树，然后重新堆化，这样重复选取值直到堆的大小为 1 为止。

&emsp;&emsp;因此，**大顶堆排序的结果是从小到大排序**，**小顶堆的排序结果是从大到小排序**。

```js
function heapSort(heap) {
    let heapSize = heap.length - 1;

    buildHeap(heap, heapSize);

    for (let i = heapSize; i > 1; --i) {
        swap(heap, 1, i);
        heapify(heap, i - 1, 1);
    }
}
```



# 8. 数组去重

```js
let inarr1 = [1, 2];
let inarr2 = {a: 'a', b: 'b'};
let arr = [1, 2, inarr1, 1, 5, inarr1, 2, 1, inarr2, inarr2];
let arr1 = [];

// 1.使用 set
arr1 = [...new Set(arr)];

// 2.使用 filter
arr1 = arr.filter((val, idx) => {
    return arr.indexOf(val) === idx;
});

// 3. 使用循环
for (let val of arr) {
    if (arr1.indexOf(val) === -1) {
        arr1.push(val);
    }
}

// 4. 排序后去重
arr.sort();
for (let val of arr) {
    if (arr1.length === 0 || arr1[arr1.length-1] !== val) {
        arr1.push(val);
    }
}
```



# 9. 随机字符串

&emsp;&emsp;主要使用 `Random` 和 `toString(radix)` 来生成随机字符串

&emsp;&emsp;`toString` 方法能接收一个参数作为进制，若进制大于 `10` ，则会使用字母来进行数字表示，且范围为 `2 ~ 36`，因此可以使用 `toString(36)` 来生成一个由数字（0 ~ 9）和字母（a ~ z）表示的字符串。

&emsp;&emsp;不过，这种方法生成的随机字符串并不安全，不能用于实际开发中。

```js
function generateRandomString(len) {
    let randomStr = '';
    for (; randomStr.length < len; randomStr += Math.random().toString(36).substr(2)) {}

    return randomStr.substr(0, len);
}
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210223173951760.png" alt="image-20210223173951760" style="zoom:80%;" />

# 10. 解析URL

ref : https://juejin.cn/post/6902060047388377095#heading-26

&emsp;&emsp;URL 的组成，常见的 url 部分使用虚线包裹：

![image-20210224181025162](https://gitee.com/ylea/imagehost/raw/master/img/image-20210224181025162.png)


举个 🌰：`https://keith:miao@www.foo.com:80/file?test=3&miao=4#heading-0`



&emsp;&emsp;一个最简单的 url 解析方法：使用 [URL](https://developer.mozilla.org/zh-CN/docs/Web/API/URL) 构造函数解析：

```js
function parseURL(url) {
    const urlObj = new URL(url);
    return {
        protocol: urlObj.protocol,
        username: urlObj.username,
        password: urlObj.password,
        hostname: urlObj.hostname,
        port: urlObj.port,
        pathname: urlObj.pathname,
        search: urlObj.search,
        hash: urlObj.hash
    };
}
```



# 11. jsonp

## JSONP 介绍

&emsp;&emsp;JSONP 即 JSON with Padding。它是借助了 HTML 中 `src` 属性可跨域的特点来进行 `GET` 请求，我们通过 `src` 中 `url` 部分的参数传递，给出回调的函数名，如 `?callback=cbFunction`，然后响应中返回要执行的 `JS` 代码，使用字符串拼接的方式传递回调函数中的参数，然后浏览器接受响应，就直接执行了返回的代码。



&emsp;&emsp;因此 JSONP 由两部分组成：**回调函数** 和 **数据**。回调函数是当响应到来时要在页面中调用的函数，函数名一般通过请求 url 传递；而数据通过服务器响应请求时通过字符串拼接作为回调函数的参数返回。





JSONP 优点：

- 不受同源策略的限制
- 兼容性好
- 实现简单

缺点：

- 只支持 `GET` 请求
- 不能解决不同域的两个页面之间如何进行 JS 调用的问题（需要调用的 JS 代码在一个域中，而获取的数据在另一个域中？）
- 属于嵌入式脚本，有一定安全风险



简例：

前端：

```html
<body>
    <div><button>点击我发送get请求</button></div>
    <!-- 写法1： -->
    <!-- <script>
        function jsonpcallback(response){
        	alert(response.message);
        }
    </script>
    <script src="http://127.0.0.1:3000/get?callback=jsonpcallback" type="text/javascript"></script> -->
</body>

<!-- 写法2： -->
<script>
    document.getElementsByTagName('button')[0].addEventListener('click', function () {
        ajax('http://127.0.0.1:3000/get', function(response){
            alert(response.message);
        });
    });
    function ajax(url, callback){
        var jsonp=document.createElement('script');
        jsonp.type = 'text/javascript';
        jsonp.src=url+'?callback=jsonpcallback';
        jsonpcallback = function(response){
            callback(response);
        };
        document.getElementsByTagName('head')[0].appendChild(jsonp);
    }
</script>

```

服务端代码：

```js
var Koa = require('koa');
var Router = require('koa-router');
var querystring = require('querystring');
var app = new Koa();
var router = new Router();

//处理get请求
router.get('/get', async function(ctx){
    var params = querystring.parse(ctx.request.url.split('?')[1]);
    var data = { message: "我是" + ctx.request.header.host + "，我收到了你的get请求！！！" }
    ctx.status=200;
    ctx.body=params['callback']+'('+JSON.stringify(data)+');';
});

app
    .use(router.routes())
    .use(router.allowedMethods())

app.listen(3000);
```



简单总结就是：

```js
function jsonpcallback(response){
    alert(response.message);
}
jsonpcallback({"message":"我是127.0.0.1:3000，我收到了你的get请求！！！"});
```



## 手写 JSONP

&emsp;&emsp;手写 jsonp 也就是通过给定的参数和回调函数名称构造一个 url，然后再创建一个 `script` 标签来使用这个 url，最后再通过回调函数获取数据：

```js
const jsonp = ({url, params, callbackName}) => {
    const generateURL = () => {
        let dataStr = '';

        for (let key in params) {
            dataStr += `${key}=${params[key]}&`;
        }
        dataStr += `callback=${callbackName}`

        return `${url}?${dataStr}`;
    };

    return new Promise((resolve, reject) => {
        callbackName = callbackName || Math.random().toString();
        let scriptEle = document.createElement('script');
        scriptEle.src = generateURL();
        document.body.appendChild(scriptEle);

        // 随后，服务器返回字符串 'callbackName(data)'，浏览器解析即可执行
        // 在全局对象中添加处理回调方法
        window[callbackName] = (data) => {
            resolve(data);
            // 最后再移除不需要的 dom 节点，防止内存泄漏
            document.body.removeChild(scriptEle);
        };
    });
};
```

简例：

```js
const jsonp = ({url, params, callbackName}) => {
    const generateURL = () => {
        let dataStr = '';

        for (let key in params) {
            dataStr += `${key}=${params[key]}&`;
        }
        dataStr += `jsoncallback=${callbackName}`
        console.log(`${url}?${dataStr}`)
        return `${url}?${dataStr}`;
    };

    return new Promise((resolve, reject) => {
        callbackName = callbackName || Math.random().toString();
        let scriptEle = document.createElement('script');
        scriptEle.src = generateURL();

        document.body.appendChild(scriptEle);

        // 随后，服务器返回字符串 'callbackName(data)'，浏览器解析即可执行
        // 在全局对象中添加处理回调方法
        window[callbackName] = (data) => {
            console.log('data: ', data);
            resolve(data);
            // 最后再移除不需要的 dom 节点，防止内存泄漏
            document.body.removeChild(scriptEle);
        };
    });
};

jsonp({
    url: 'https://www.runoob.com/try/ajax/jsonp.php',
    callbackName: 'callbackFunction'
}).then((data) => {
    console.log(data)
});
```





# 12. 图片懒加载

&emsp;&emsp;懒加载是一种网页性能优化策略，若一个页面中当前不可见区域有图片需要加载，那么可以延迟该图片的请求，直到达到该图片的可视区域为止。

## 懒加载的实现原理

&emsp;&emsp;网页中一般占用较多资源的是图片文件，因此懒加载一般是针对图片而言的。

&emsp;&emsp;而一张图片就是一个 `<img>` 标签，图片的加载源就是其 `src` 属性，浏览器发起请求就是根据其是否有 `src` 属性来决定的。

&emsp;&emsp;因此可以根据 `<img>` 标签的位置，当标签在可视区域之外时，其 `src` 属性为空，而当即将进入可视区域时，设置其 `src` 属性，让浏览器发起图片请求。

&emsp;&emsp;所以说**懒加载的要点就是可视区域的判断**

&emsp;&emsp;在页面进行滚动时调用判断函数，因此**需要进行防抖优化**



&emsp;&emsp;首先，用一幅图解释浏览器的宽高：

![Dimensions-client](https://gitee.com/ylea/imagehost/raw/master/img/Dimensions-client.png)

## 方法1：clientHeight、scrollTop 和 offsetTop

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210224181307061.png" alt="image-20210224181307061" style="zoom: 67%;" />

&emsp;&emsp;由上图可知，页面的高度为 `clientHeight`，若有滚轮的话，`scrollTop` 会是当前页面滚动的高度，而 `offsetTop` 会是元素距离页面顶部的高度。

&emsp;&emsp;因此当 `scrollTop + clientHeight = offsetTop` 时，就说明元素即将进入可视区域，图片需要进行请求了。

> **`HTMLElement.offsetTop`** 为只读属性，它返回当前元素相对于其 [`offsetParent`](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLElement/offsetParent) 元素的顶部内边距的距离。
>
> **`Element.scrollTop`** 属性可以获取或设置一个元素的内容垂直滚动的像素数。
>
> **`HTMLElement.clientHeight`** 对于没有定义CSS或者内联布局盒子的元素为0，否则，它是元素内部的高度(单位像素)，包含内边距，但**不包括**水平滚动条、边框和外边距。
>
> `clientHeight` 可以通过 CSS `height` + CSS `padding` - 水平滚动条高度 (如果存在)来计算.
>
> `clientHeight` 可以使用 `window.innerHeight` 来替代 ，不过若有水平滚动条，`innerHeight` 也会包括滚动条高度
>
> 
>
> `document.documentElement` 总是会返回一个 [`html`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/html) 元素，且它一定是该文档的根元素。

```js
function getTop(e) {
    let T = e.offsetTop;
    while (e = e.offsetParent) {
        T += e.offsetTop;
    }

    return T;
}

function lazyLoad(imgs) {
    // 视口高度
    let viewHeight = document.documentElement.clientHeight;
    // 滚动条拉伸的高度
    let scrollTop = document.documentElement.scrollTop  || document.body.scrollTop ;

    for (let i = count; i < imgs.length; ++i) {
        if (getTop(imgs[i]) <= viewHeight + scrollTop) {
            if (!imgs[i].getAttribute('src')) {
                imgs[i].src = imgs[i].getAttribute('data-src');
                count++;
            }
        }
    }
}
```

## 方法2： 使用 getBoundingClientRect 方法

[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)

> **`Element.getBoundingClientRect()`**  方法返回元素的大小及其相对于视口的位置。
>
> 
>
> 如果是标准盒子模型，元素的尺寸等于`width/height` + `padding` + `border-width`的总和。如果`box-sizing: border-box`，元素的的尺寸等于 `width/height`。
>
> 
>
> 返回的结果是包含完整元素的最小矩形，并且拥有`left`, `top`, `right`, `bottom`, `x`, `y`, `width`, 和 `height`这几个以像素为单位的只读属性用于描述整个边框。除了`width` 和 `height` 以外的属性是相对于**视图窗口的左上角**来计算的。

<img src="https://gitee.com/ylea/imagehost/raw/master/img/rect.png" alt="rect" style="zoom: 50%;" />

&emsp;&emsp;因此 DOM 元素的 `getBoundingClientReact().top` 属性就能直接判断图片是否出现在了当前视口中。

```js
function lazyLoad(imgs) {
    let viewHeight = document.documentElement.clientHeight;

    for (let i = count; i < imgs.length; ++i) {
        if (imgs[i].getBoundingClientRect().top <= viewHeight) {
            if (!imgs[i].getAttribute('src')) {
                imgs[i].src = imgs[i].getAttribute('data-src');
                count++;
            }
        }
    }
}
```

## 方法3：IntersectionObserver

[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)

[阮一峰 IntersectionObserver 教程](http://www.ruanyifeng.com/blog/2016/11/intersectionobserver_api.html)

&emsp;&emsp;`IntersectionObserver` 浏览器内置的 API，实现了监听 window 的 `scroll` 事件、`判断是否在视口中` 以及 `节流` 三大功能。该 API 需要 polyfill。



下面的代码中

- 传入的回调函数的 `changes` 是[`IntersectionObserverEntry`](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserverEntry) 对象的数组，数组中的每个对象的 `target` 是实际可视情况发生改变的 `dom` 元素
- `getElementsByTagName` 返回的是一个 `HTMLCollection`，因此需要使用 `Array.from` 来转换一下

```js
let imgs = document.getElementsByTagName('img');

const observer = new IntersectionObserver(changes => {
    changes.forEach(dom => {
        if (dom.isIntersecting) {
            dom.target.src = dom.target.getAttribute('data-src');
            // 当加载后就从监视器中移除
            observer.unobserve(dom.target);
        }
    });
});

Array.from(imgs).forEach(dom => observer.observe(dom));
```



## 方法4：使用开源库

https://github.com/aFarkas/lazysizes

## 小结

&emsp;&emsp;上面的前三种方法，前两种方法是有一定缺陷的，它们只比较了距离页面顶端的距离，因此若刷新浏览器时，所处位置的上方所有可见和不可见的图片都会被加载出来，因此并不是完备的懒加载。

&emsp;&emsp;不过最后一种使用 `IntersectionObserver` 的方法则不会有这个问题，它唯一的弱势是观察器在进程中的优先级会比较低。



## 实例

```html
<head>
    <style>
        body {
            height: 3000px;
        }
        img {
            width: 500px;
            height: 500px;
            object-fit: scale-down;
        }
    </style>
</head>
<body>
    <img data-src="./imgs/1.png">
    <img data-src="./imgs/2.png">
    <img data-src="./imgs/3.png">
    <img data-src="./imgs/4.png">
    <img data-src="./imgs/5.png">
    <img data-src="./imgs/6.png">
    <img data-src="./imgs/7.png">
    <img data-src="./imgs/8.png">
<script>
    // 方式1：clientHeight、scrollTop、offsetTop
    let imgs = document.getElementsByTagName('img'), count = 0;

    // offsetTop 是元素与 offsetParent 的距离，循环获取直到达到页面顶部
    function getTop(e) {
        let T = e.offsetTop;
        while (e = e.offsetParent) {
            T += e.offsetTop;
        }

        return T;
    }

    function lazyLoad(imgs) {
        // 视口高度
        let viewHeight = document.documentElement.clientHeight;
        // 滚动条拉伸的高度
        let scrollTop = document.documentElement.scrollTop  || document.body.scrollTop ;

        for (let i = count; i < imgs.length; ++i) {
            if (getTop(imgs[i]) <= viewHeight + scrollTop) {
                if (!imgs[i].getAttribute('src')) {
                    imgs[i].src = imgs[i].getAttribute('data-src');
                    count++;
                }
            }
        }
    }

    function throttle(fn, wait, ...args) {
        let activeTime = 0;

        return function() {
            let current = +new Date();
            if (wait <= current - activeTime) {
                fn.apply(this, args.concat(...arguments));
                activeTime = current;
            }
        };
    }
    // 首次加载
    window.onload = lazyLoad(imgs);
    window.onscroll = throttle(lazyLoad, 100, imgs);
</script>
</body>
```

