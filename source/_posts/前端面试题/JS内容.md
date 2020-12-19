---
title: JS内容
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/js.jpg'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/63.jpg'
date: 2020-12-13 22:16:24
categories:
    - 前端面试题
tags:
    - JS 面试题
---

# 1. ['1', '2', '3'].map(parseInt)

&emsp;&emsp;这题涉及到两个函数，[Array.prototype.map](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array/map) 和 [parseInt](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/parseInt)

说点题外话，首先，`map` 函数的调用不会改变原数组，它会返回一个新数组，其次，map 函数在调用回调时的处理元素的范围已经确定了，过程中再追加的数组元素不会被访问到。



&emsp;&emsp;回到本题，`map` 函数的第一个参数是一个回调函数，第二个参数是可选的 `this` 指向，回调函数会被**自动传入三个参数**：**数组元素**、**元素索引**、原数组本身。

&emsp;&emsp;而 `parseInt` 函数接收两个参数，第一个是要被解析的值，第二个是可选的进制基数（ `2 - 36`，当第二个参数为 `undefined` 或 `0` 时，若第一个参数以 `0x` 开头，会认为是十六进制，以 `0` 开头会认为是八进制，其他开头会默认为十进制）。

&emsp;&emsp;当传入的第二个参数小于 `2` 或大于 `36` 时，函数会返回 `NaN`



&emsp;&emsp;这样问题就来了，在 `map` 函数中使用 `parseInt` 作为回调函数，会自动传入数组元素和元素索引作为 `parseInt` 函数的两个参数，因此解析的过程就是：

```js
parseInt('1', 0); // 1
parseInt('2', 1); // NaN
parseInt('3', 2); // NaN
```

&emsp;&emsp;根据上面对 `parseInt` 的描述，因为 `radix` 为 `0` 且不是 `0x` 或 `0` 开头，因此默认为十进制，返回 `1`；

&emsp;&emsp;第二行，因为 `radix = 1 < 2`，因此返回 `NaN`；

&emsp;&emsp;第三行，`radix = 2`，但 `3` 超出了二进制范围，因此也返回 `NaN`。



&emsp;&emsp;同理，`['10','10','10','10','10'].map(parseInt);` 返回值为 `[10, NaN, 2, 3, 4]`



&emsp;&emsp;那么要怎么修改才能使结果变成我们想要的答案呢？

&emsp;&emsp;1.使用 `Number` 构造函数：

```js
['1', '2', '3'].map(Number); // [1, 2, 3]
```

&emsp;&emsp;2.`map` 中指定 `parseInt` 函数的参数：

```js
['1', '2', '3'].map(v => parseInt(v)); // 或 parseInt(v, 10)，指定为 10 进制
```



## 一个变形

&emsp;&emsp;使用了连续的箭头表达式，指定了`map` 回调中的参数

```js
let unary = fn => val => fn(val)
let parse = unary(parseInt)
console.log(['1.1', '2', '0.3'].map(parse))
```

&emsp;&emsp;相当于：

```js
['1.1', '2', '0.3'].map((val, idx, arr) => parseInt(val));
```



# 2. 什么是防抖和节流，有什么区别

REF :

https://github.com/mqyqingfeng/Blog/issues/22

https://github.com/mqyqingfeng/Blog/issues/26

## 前言

&emsp;&emsp;对于一些频繁的操作，如对窗口的 `resize`、`scroll`、输入框内容改动响应时，如果相应处理函数没有频率限制的话，会加重浏览器的负担，导致用户体验差，而防抖(debounce) 和节流(throttle) 可以有效减少处理函数的调用频率，同时不影响实际效果。

## 防抖

&emsp;&emsp;触发高频事件后 `n` 秒内函数只会执行一次，如果 `n` 秒内高频事件再次被触发，则重新计算时间。

&emsp;&emsp;一个连续操作中的处理，只触发一次，从而实现防抖动。

> 思路：每次触发事件后都取消之前的延迟调用的方法

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

### 立即执行的 debounce

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



### 带返回值的 debounce

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

### 可取消的 debounce

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











## 节流

&emsp;&emsp;高频事件触发，但在 `n` 秒内只会执行一次，所以节流会稀释函数的执行频率。



&emsp;&emsp;一个连续操作中的处理，按照阀值时间间隔进行触发，从而实现节流。

&emsp;&emsp;在一定的时间间隔内，某个事件之会执行一次

> 思路：每次触发事件时都判断当前是否有在等待执行的回调函数

### 使用时间戳

&emsp;&emsp;根据时间戳来判断两次响应的间隔是否大于设置的间隔，大于才能执行下一次响应函数。

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

### 使用定时器

使用了定时器，当事件触发时不会立即执行，等待时间过后才会开始执行，然后清空计数器。

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



### 比较两个节流方法

1. 使用时间戳的节流方法触发事件时会立即执行，而使用定时器的方法会在触发事件 `n` 秒后再执行；
2. 使用时间戳的方法停止触发事件后不会再执行，使用定时器的方法会在停止触发事件的 `n` 秒后再执行一次。

### 综合节流方法

综合时间戳和定时器的效果，当触发事件时会立即执行，且在停止事件触发的 `n` 秒后还会再执行一次。

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
            // 若是调整了系统时间导致的立即执行，若前面有设置一个 timer，需要将其清除
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

### 通过参数调整节流效果

有时候我们希望节流效果为 “有头无尾”， 有时候希望是 “有尾无头”。

这样的话我们可以通过设置参数 `options` 来调整，约定：

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

### 设置取消功能

与防抖中相似，当节流中需要等待很久才能进行下一次响应时，若想提前响应，可以使用 `cancel` 来取消：

```js
throttled.cancel = function() {
    clearTimeout(timeout);
    previous = 0;
    timer = null;
};
```

### 注意事项

在上面的带有 `options` 参数的实现中，`leading：false` 和 `trailing: false` 不能同时设置。

若同时设置的话，当将鼠标移出时，因为 `trailing` 设置为 `false` ，停止触发的时候不会设置定时器，所以只要再过了设置的时间，再移入的话，就会立即执行了，这就违反了 `leading: false` 的规则。因此，这个 `throttle` 只有三种使用方法：

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



## 场景

- a、`scroll`事件：当页面发生滚动时，`scroll`事件会被频繁的触发，`1s`触发可高达上百次。在`scroll`事件中，如果有复杂的操作（特别是影响布局的操作），将会大大影响性能，甚至导致浏览器崩溃。所以，对其进行防抖、限频很重要。
- b、`click`事件：用户进行`click`事件时，有可能连续触发点击（用户本意并非双击）。该操作有可能是不小心多次连续点击，也可能是页面状况不好的情况下，期待尽快得到反馈的有意行为；但这样的操作，反而会加剧性能问题，因此也有必要考虑防抖、限频。
- c、`input`事件：如`sug`等需要通过`ajax`及时获得数据的情况，需要进行限频，防止频繁的请求发生，减少服务器压力的同时，提高页面响应性能。
- d、`touchmove`事件：同`scroll`事件类似。



### 防抖

&emsp;&emsp;示例：有一个 `input` 框，当在 `input` 框中输入时，会调用 `sayHi` 函数打印一句话，而对于输入框来说，输入会很频繁，如连续输入 `12345`，这样会调用 `5` 次 `sayHi` 函数，但实际上我们只想输入一串 `12345` 并调用一次，这时 `debounce` 函数就派上用场了，当连续输入时，输入的间隔会很小，因此若在规定的时间间隔内再次输入，会重新计时，然后最后当输入结束时在调用一次 `sayHi`。

&emsp;&emsp;诸如此类的事件还有：当页面的滚动条发生滚动 & 当前窗口的大小发生改变 & 当输入框的内容发生改变....

```js
function sayHi() {
    console.log('debounce success!');
}

let inp = document.createElement('input');
document.body.appendChild(inp);

inp.addEventListener('input', debounce(sayHi));
```



### 节流

&emsp;&emsp;示例：当窗口进行缩放时，一秒内只会执行一次打印窗口大小的函数。

```js
function sayHi(e) {
    console.log(e.target.innerWidth, e.target.innerHeight);
}

window.addEventListener('resize', throttle(sayHi));
```







# 3. this 指向问题

&emsp;&emsp;试说出程序的输出：

```js
var number = 10;

function fn() {
    console.log(this.number);
}

var obj = {
    number: 2,
    show: function(fn) {
        this.number = 3;
        fn();
        arguments[0]();
    }
};

obj.show(fn);
```

&emsp;&emsp;首先，在 `obj` 中调用了 `show` 函数，因此 `show` 函数中的 `this` 指向会是 `obj`，此时， `show` 函数调用了传入的 `fn` 函数，`fn` 函数是位于全局变量中的，因此 `fn` 中的 `this` 指向了全局变量，`console.log()` **输出 `10`**，若想要正确打印 `3`，那么可以修改为：`fn.bind(this)();`

&emsp;&emsp;然后，`arguments` 为 `show` 函数中的参数列表，第一个参数是 `fn`（调用相当于是 `arguments.0()`，类似于 `obj.show`），因此此时 `fn` 中的 `this` 是指向 `arguments` 对象的，但 `arguments` 对象中没有定义 `number` 变量，因此**输出 `undefined`**。



&emsp;&emsp;稍微修改一下：

```js
function fn() {
    console.log('this in fn: ', this);
}

var obj = {
    show: function(fn) {
        console.log('this in show: ', this);
        fn();
        arguments[0]();
    }
};

obj.show(fn);
```

&emsp;&emsp;根据我们上面的分析，打印的结果会是：

1.`this in show:` + `obj 对象`

2.`this in fn:` + `全局对象`

3.`this in fn:` + `arguments 对象`



# promise 及 setTimeout 执行顺序

&emsp;&emsp;有代码如下：

```js
setTimeout(function() {
    console.log(1);
}, 0);

new Promise(function(res, rej) {
    res(2);
    console.log(0); 
}).then(console.log);

console.log(3);
```

&emsp;&emsp;首先，对于 `setTimeout` 任务来说，它会被排到执行队列的尾部，同步任务执行完后立即执行这个任务；

&emsp;&emsp;而 `promise` 一旦建立，其中的任务就会立即执行，因此会**先输出 `0`**，然后是最外层的 `console.log(3)` 为同步任务，会按顺序执行，因此**第二个输出 `3`**，然后再执行 `promise.then` ，即打印 `2`。

因此打印顺序为：`0 -> 3 -> 2 -> 1`。



## 微任务和宏任务

&emsp;&emsp;微任务包括 `process.nextTick`、`promise`、`Object.observe`、`MutationObserver`

&emsp;&emsp;宏任务包括 `script`、`setTimeout`、`setInterval`、`setImmediate`、`I/O`、`UI rendering`

&emsp;&emsp;并不是所有的微任务都快于宏任务，因为宏任务中包括了 `script`，浏览器会先执行一个宏任务，接下来有异步代码的话再执行微任务。

&emsp;&emsp;所以一次正确的 `Event Loop` 是这样的：

1. 执行同步代码，这属于宏任务
2. 执行栈为空，查询是否有微任务需要执行
3. 执行所有的微任务
4. 必要的话渲染 UI
5. 然后开始下一轮的 `Event Loop`，执行宏任务中的异步代码

&emsp;&emsp;通过上面的执行步骤可知，**如果宏任务中的异步代码有大量的计算且需要操作 DOM 的话，为了更快的界面响应，可以把操作放入微任务中**。



# ES5/ES6 的继承除了写法上还有什么差异

在 ES6 中，子类可以通过 `__proto_` 寻址到父类，而通过 ES5 的方式，`Sub.__proto__ === Function.prototype`。

**ES6：**

```js
class Super {}

class Sub extends Super {}

const sub = new Sub();

console.log(Sub.__proto__ === Super); // true
```

**ES5：**

```js
function Super() {}

function Sub() {}

Sub.prototype = new Super();

Sub.prototype.constructor = Sub;

const sub = new Sub();

console.log(Sub.__proto__ === Function.prototype); // true
```





































