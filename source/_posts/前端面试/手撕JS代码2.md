---
title: 手撕JS代码2
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/js.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/44.png'
hide: true
date: 2021-02-24 17:25:06
categories:
tags:
---



# 1. Promise

## Promise 基本实现

实现了 Promise 的基本方法 `then`、`catch` 和 `finally`

```js
class MyPromise {
    constructor(fn) {
        this.state = 'pending';
        this.value = null;
        this.callbacks = [];

        // 初始化时，调用传入的 fn 回调函数，并将 _resolve 和 _reject 方法传入
        fn(this._resolve, this._reject);
    }

    catch = (onError) => {
        return this.then(null, onError);
    };

    finally = (onDone) => {
        return this.then(onDone, onDone);
    };
	
	// then 方法返回一个新的 Promise 实例作为桥接对象，将传入的回调和桥接Promise的 resolve、reject 方法注册到当前 Promise 对象中，这是 Promise 链式调用的关键
    then = (onFulfilled, onRejected) => {
        return new MyPromise((resolve, reject) => {
            this._handle({
                onFulfilled,
                onRejected,
                resolve,
                reject
            });
        });
    };

    _handle = (callback) => {
        // 若处于 pending 状态，将注册的回调保存下来
        if (this.state === 'pending') {
            this.callbacks.push(callback);
            return;
        }

        let cb = this.state === 'fulfilled' ? callback.onFulfilled : callback.onRejected;
        let next = this.state === 'fulfilled' ? callback.resolve : callback.reject;
		
        // 若没有传入处理回调，则直接调用对应状态的处理函数
        if (!cb) {
            next(this.value);
            return;
        }

        try {
            let ret = cb(this.value);
            // 调用注册的桥接对象的 resolve 方法，若 ret 是一个 Promise 对象，那么桥接 Promise 的状态会由 ret 决定，否则对于其他类型的返回值，直接 resolve
            callback.resolve(ret);
        } catch (error) {
            callback.reject(error);
        }
    };

    _resolve = (value) => {
        if (this.state !== 'pending') return;

        if (this._link(value)) return;

        this.state = 'fulfilled';
        this.value = value;

        setTimeout(this._handleCb, 0);
    };

    _reject = (reason) => {
        if (this.state !== 'pending') return;

        if (this._link(reason)) return;

        this.state = 'rejected';
        this.value = reason;

        setTimeout(this._handleCb, 0);
    };

    _handleCb = () => {
        this.callbacks.forEach((callback) => {
            this._handle(callback);
        });
    };
	
	// 若传入的是一个 thenable 对象，那么将当前 promise 的 resolve 和 reject 方法作为回调注册到这个对象中
    _link = (p) => {
        if (p && (typeof p === 'object' || typeof p === 'function')) {
            const then = p.then;
            if (typeof then === 'function') {
                then.call(p, this._resolve, this._reject);
                return true;
            }
        }

        return false;
    };
}
```







## Promise 静态方法

### Promise.resolve

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/resolve)

`Promise.resolve` 方法根据传入的参数来返回一个 Promise 实例。

- 若传入的参数是一个 Promise 对象，那么直接返回这个 Promise 对象。
- 若传入的参数是一个 `thenable` 对象（带有 `then` 方法的对象），那么会调用其 `then` 方法，并且最终状态由这个对象决定。
- 否则，返回一个 `fulfilled` 状态的 Promise 对象。

```js
// 静态方法 resolve，根据传入的参数返回一个 promise 对象
static resolve(value) {
    // 若传入的参数是 Promise 类型，那么直接返回
    if (value instanceof MyPromise) {
        return value;
    } else if (value && (typeof value === 'object' || typeof value === 'function')) {
        // 若传入的参数是一个 thenable 类型的对象，那么调用其 then 方法，并返回一个新的 promise 对象
        const then = value.then;
        if (typeof then === 'function') {
            return new MyPromise((resolve, reject) => {
                then.call(value, resolve, reject);
            });
        }
    } else {
        // 否则创建一个状态为 fulfilled 的 promise 对象，并将 value 作为参数传入
        return new MyPromise(resolve => resolve(value));
    }
}
```



### Promise.reject

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/reject)

返回一个 `rejected` 状态的 Promise 对象

```js
static reject(value) {
    return new MyPromise((resolve, reject) => {
        reject(value);
    });
}
```

### Promise.all

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/all)

`Promise.all` 方法接收一个 `promise` 的 `iterable` 类型的参数，并返回一个 Promise 实例。

- 当其中所有 Promise 类型的对象都 `resolve` 完毕，则返回所有回调的执行结果
- 若其中有一个 promise 被 `rejected` 或者程序执行出错，则抛出错误，且若是因为 `reject`，那么错误信息就是 `reject` 回调的执行结果。

```js
// 传入的是一个 iterable 对象，其中的元素可能不会全是 promise 对象
// 可以调用 Promise 将不是 promise 的对象转换成一个 Promise 实例，也可以直接对其进行结果保存
static all(promises) {
    return new MyPromise((resolve, reject) => {
        let res = [];
        promises = Array.from(promises);    // 传入的对象是 iterable 对象，需要先统一转换为 Array
        let len = promises.length;
        if (len === 0) return resolve(promises);

        const handle = (promise, i) => {
            try {
                // 通过该 promise 的 then 方法来注册一个 onFulfilled 回调，onFulfilled 中调用 handle 方法来进行执行结果的传递
                if (promise && (typeof promise === 'object' || typeof promise === 'function')) {
                    const then = promise.then;
                    if (typeof then === 'function') {
                        then.call(promise, function(val) {
                            handle(i, val);
                        }, reject);
                    }
                    return;
                }

                res[i] = promise;
                if (--len === 0) {
                    resolve(res);
                }
            } catch (error) {
                reject(error);
            }
        };

        promises.forEach((promise, i) => {
            handle(i, promise);
        });
    });
}
```



### Promise.race

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/race)

> **`Promise.race(iterable)`** 方法返回一个 promise，一旦迭代器中的某个 promise 解决或拒绝，返回的 promise 就会解决或拒绝。

调用所有 Promise 实例的 `then` 方法，注册 `resolve` 和 `reject` 回调，对于非 Promise 实例，直接调用 `resolve`。

```js
static race(promises) {
    return new MyPromise((resolve, reject) => {
        promises.forEach((promise) => {
            if (promise instanceof MyPromise) {
                promise.then(resolve, reject);
            } else {
                resolve(promise);
            }
        });
    });
}
```



## 完整的 Promise

```js
class MyPromise {
    constructor(fn) {
        this.state = 'pending';
        this.value = null;
        this.callbacks = [];

        fn(this._resolve, this._reject);
    }

    static resolve(value) {
        if (value instanceof MyPromise) {
            return value;
        } else if (value && (typeof value === 'object' || typeof value === 'function')) {
            const then = value.then;
            if (typeof then === 'function') {
                return new MyPromise((resolve, reject) => {
                    then.call(value, resolve, reject);
                });
            }
        } else {
            return new MyPromise(resolve => resolve(value));
        }
    }

    static reject(value) {
        return new MyPromise((resolve, reject) => {
            reject(value);
        });
    }

    static all(promises) {
        return new MyPromise((resolve, reject) => {
            let res = [];
            promises = Array.from(promises); 
            let len = promises.length;
            if (len === 0) return resolve(promises);

            const handle = (i, promise) => {
                try {
                    if (promise && (typeof promise === 'object' || typeof promise === 'function')) {
                        const then = promise.then;
                        if (typeof then === 'function') {
                            then.call(promise, function(val) {
                                handle(i, val);
                            }, reject);
                        }
                        return;
                    }

                    res[i] = promise;
                    if (--len === 0) {
                        resolve(res);
                    }
                } catch (error) {
                    reject(error);
                }
            };

            promises.forEach((promise, i) => {
                handle(i, promise);
            });
        });
    }

    static race(promises) {
        return new MyPromise((resolve, reject) => {
            promises.forEach((promise) => {
                if (promise instanceof MyPromise) {
                    promise.then(resolve, reject);
                } else {
                    resolve(promise);
                }
            });
        });
    }

    catch = (onError) => {
        return this.then(null, onError);
    };

    finally = (onDone) => {
        return this.then(onDone, onDone);
    };

    then = (onFulfilled, onRejected) => {
        return new MyPromise((resolve, reject) => {
            this._handle({
                onFulfilled,
                onRejected,
                resolve,
                reject
            });
        });
    };

    _handle = (callback) => {
        if (this.state === 'pending') {
            this.callbacks.push(callback);
            return;
        }

        let cb = this.state === 'fulfilled' ? callback.onFulfilled : callback.onRejected;
        let next = this.state === 'fulfilled' ? callback.resolve : callback.reject;

        if (!cb) {
            next(this.value);
            return;
        }

        try {
            let ret = cb(this.value);
            callback.resolve(ret);
        } catch (error) {
            callback.reject(error);
        }
    };

    _resolve = (value) => {
        if (this.state !== 'pending') return;

        if (this._link(value)) return;

        this.state = 'fulfilled';
        this.value = value;

        setTimeout(this._handleCb, 0);
    };

    _reject = (reason) => {
        if (this.state !== 'pending') return;

        if (this._link(reason)) return;

        this.state = 'rejected';
        this.value = reason;

        setTimeout(this._handleCb, 0);
    };

    _handleCb = () => {
        this.callbacks.forEach((callback) => {
            this._handle(callback);
        });
    };

    _link = (p) => {
        if (p && (typeof p === 'object' || typeof p === 'function')) {
            const then = p.then;
            if (typeof then === 'function') {
                then.call(p, this._resolve, this._reject);
                return true;
            }
        }

        return false;
    };
}
```











































































