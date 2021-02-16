---
title: JS 函数
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/js.jpg'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/62.jpg'
date: 2020-11-27 16:16:34
categories:
    - JS
tags:
    - JS Function
---


# 函数

## 函数也是对象

```js
function bark() {
  console.log('Woof!');
}

bark.animal = 'dog';
```

&emsp;&emsp;上面的代码在 JS 中是可行的，因为函数也是对象（**除了基本类型之外其他都是对象**）。

## 构造器函数

&emsp;&emsp;在 JS 中，结合 `new` 前缀来调用的函数被称为构造器函数，**构造器函数按约定命名的首字母必须大写。**

```js
function Person(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
}

const member = new Person('Lydia', 'Hallie');
```

&emsp;&emsp;对于构造器函数来说，**不能**像其他对象一样直接给它添加对象，**应该**使用 `prototype` 来添加，这样所有 `Person` 实例化的对象都会有这个属性了：

```js
Person.getFullName = function() {
    return `${this.firstName} ${this.lastName}`;
};
console.log(member.getFullName());  // TypeError

Person.prototype.getFullName = function() {
    return `${this.firstName} ${this.lastName}`;
};
```

### 构造器函数实例化

&emsp;&emsp;若对一个函数**使用** `new` 实例化的话，实例化对象会指向这个新创建的对象，而**不使用** `new`的话，则会指向全局对象 `global`。

```js
function Person(firstName, lastName) {
  this.firstName = firstName;
  this.lastName = lastName;
}

const lydia = new Person('Lydia', 'Hallie');
const sarah = Person('Sarah', 'Smith');

console.log(lydia);  // Person {firstName: "Lydia", lastName: "Hallie"}
console.log(sarah);  // undefined
```

&emsp;&emsp;上面的代码中，对于 `sarah` 来说，相当于定义了 `global.firstName = 'Sarah'` 与 `global.lastName = 'Smith'`，并且调用了 `Person` 函数，可是它并没定义返回值，因此默认返回 `undefined`.



&emsp;&emsp;**对于一个带返回对象的构造器函数来说，返回的对象的优先级更高.**

```js
function Car() {
    this.make = "Lamborghini";
    return { make: "Maserati" };
}

const myCar = new Car();
console.log(myCar);
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200416205516433.png" alt="image-20200416205516433" style="zoom:80%;" /> 

&emsp;&emsp;不过**返回的不是对象的话，返回的是构造出的对象**（数组也是对象，因此会优先返回数组）

```js

```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200416205600285.png" alt="image-20200416205600285" style="zoom:80%;" />

## 参数

&emsp;&emsp;**普通参数是传值**，而**对象是传递引用**，因此若在函数中对对象做出改动，原对象会跟着改动：

```js
function getInfo(member, year) {
    member.name = 'Lydia'; // 函数中改动对象的属性值
    year = '1998';
}

const person = { name: 'Sarah' };
const birthYear = '1997';

getInfo(person, birthYear);

console.log(person, birthYear);  // { name: "Lydia" }, "1997"
```



### arguments

&emsp;&emsp;当函数被调用时，会被附上一个参数，那就是 `arguments` 数组。

&emsp;&emsp;函数可以通过此参数访问所有它被调用时传递给它的**参数列表**，包括那些没有被分配给函数声明时定义的形式参数的多余参数。

```js
//构造一个将大量的值相加的函数
//该函数内部定义的 sum 不会与外部定义的 sum 产生冲突，因为该函数只会看到内部的那个变量
var sum = function () {
    var i,sum = 0;
    for(var i = 0; i < arguments.length; i++){
        sum += arguments[i];
    }
    return sum;
};
document.write(sum(4,8,15,16,23,42));   //-->108
```

&emsp;&emsp;==注意==，`arguments` 并不是一个真正的数组。它只是一个 “类似数组的对象” 。`arguments` 拥有一个 `length` 属性，但它没有任何数组的方法。



## 函数中的默认参数

&emsp;&emsp;下面的写法是正确的，若只传递一个参数，那么 `num2` 会是以已初始化的 `num1` 为值，不过若是 `num1` 在 `num2` 的右边的话，则会报错。

```js
function sum(num1, num2 = num1) {
    console.log(num1 + num2);
}

sum(10);  // 20
```

&emsp;&emsp;**默认参数可设为上下文内容中的变量**：

```js
function compareMembers(person1, person2 = person) {
    if (person1 !== person2) {
        console.log('Not the same!');
    } else {
        console.log('They are the same!');
    }
}

const person = { name: 'Lydia' };

compareMembers(person); // They are the same!
```

### 展开语法作为函数的默认参数

&emsp;&emsp;展开表达式可用于对象的复制，这种复制是**浅复制**，它们的指向是**不一样**的。

```js
const person = {
    name: 'Lydia',
    age: 21,
};

let p1 = {...person};
console.log(p1 === person); // false

const changeAge = (x = { ...person }) => {
    x.age += 1;
    if (x === person) {
        console.log('等于'); // 被打印
    } else {
        console.log('不等于');
    }
}
const changeAgeAndName = (x = { ...person }) => {
    if (x === person) {
        console.log('等于');
    } else {
        console.log('不等于');  // 被打印
    }
    x.age += 1;
    x.name = "Sarah";
}

changeAge(person);	// 等于  --> 传递了 person，因此 x 与 person 的指向相同
changeAgeAndName();	// 不等于  --> 没传递 person ，因此 x 只是 person 的浅复制
```



## Apply & Call

### apply

[apply NDM 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/apply)

&emsp;&emsp;`Function.prototype.apply(obj, args)`

&emsp;&emsp;`apply` 方法让我们构建一个参数数组传递给调用函数。它允许我们选择 `this` 的值。

&emsp;&emsp;`apply` 方法接收两个参数，第一个是要绑定给 `this` 的值，第二个就是一个**参数数组**。

&emsp;&emsp;当第一个参数为 `null` 或为 `undefined` 时，会自动替换为指向全局对象。

### call

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/call)

&emsp;&emsp;`Function.prototype.call(obj, arg1, arg2, ...)`

&emsp;&emsp;`call` 方法的参数和 `apply` 类型，不过第二个参数开始是传入一个个的参数而不是整个的参数列表。

&emsp;&emsp;同样，当第一个参数为 `null` 或为 `undefined` 时，会自动替换为指向全局对象。



### 第一个例子

&emsp;&emsp;构造一个包含两个数字的数组，并将它们相加：

```js
// 定义 add 函数
let add = function(a,b){
	return a + b;
};

let arr = [3, 4];
// 使用 apply
let sum = add.apply(null, arr); // sum = 7

// 使用 call
sum = add.call(null, arr[0], arr[1]); // sum = 7
```

&emsp;&emsp;上面的代码中，`apply` 的第一个参数为 `null`，因此此时 `add` 中的 `this` 会指向全局对象，如：

```js
var a = 6, b = 7;
let add = (a, b) => {
    return this.a + this.b;
};

let arr = [3, 4];
let sum = add.apply(null, arr); // sum = 13 ，此时 add 中为 6 + 7 , call 也是一样的
```



### 第二个例子

```js
let Person = function(name, age) {
    this.name = name;
    this.age = age;
};

function Student(name, age, grade) {
    // 使用 apply
    Person.apply(this, arguments); // arguments 是函数中的参数列表
    
    // 使用 call
    Person.call(this, name, age);
    
    this.grade = grade;
};

let stu = new Student('张三', 18, 2);
console.log('stu: ', stu.name, stu.age, stu.grade); // stu:  张三 18 2
```

&emsp;&emsp;上面的代码中，`Student` 构造器中没有定义 `name` 和 `age` 属性，不过因为使用了 `apply/call` 函数，修改了 `Person` 中 `this` 的指向，因此 `Student` 对象中仍有 `name` 和 `age`。

&emsp;&emsp;其中，`arguments` 是一个参数列表，在上面就会是这样：`["张三", 18, 2]`，它虽然有 `length` 属性，但并不是一个数字，只是一个类似数组的存在。



### apply 和 call 的使用选择

&emsp;&emsp;根据上面两个示例，我们可以知道，`apply` 和 `call` 的区别就是第一个参数后面的其他参数传递不同，`apply` 是传递一个参数数组，而 `call` 是传递一个个单独的参数，因此，若函数定义的参数顺序和我们的参数列表中的顺序一致，可以选择 `apply`，否则，我们可以使用 `call` 并一个个指定要传递的参数。

&emsp;&emsp;不过讲道理，使用 `call` 一个个指定传递的参数顺序的话其实将这些参数构造成数组并使用 `apply` 传递并没有什么区别。如 `call(null, 3, 4)` 和 `apply(null, [3, 4])`。



### apply 的一些妙用

&emsp;&emsp;从上面两个例子中，我们可以知道，`apply` 的第二个参数是一个参数列表，但是在实际调用的时候这个参数列表会被拆解为一个个的参数，如上面 `add` 方法中传入 `[3, 4] -> 3, 4` ，借助 `apply` 的这种特性，有一些奇妙的使用方法：

#### Math.max / min + apply

&emsp;&emsp;`Math.max/min` 不支持 `Math.max/min([param1, param2, ...])` 这样的数组参数，但它支持 `Math.max/min(param1, param2, ...)` 这样的参数，因此可以使用 `apply` 将数组拆分为一个个的参数：

```js
Math.max.apply(null, [1, 2, 3, 4]);
```

#### array.push

&emsp;&emsp;`push` 也可以通过 `apply` 来便捷的合并数组：

```js
let arr1 = [1, 2], arr2 = [3, 4];
Array.prototype.push.apply(arr1, arr2);

console.log(arr1); // [1, 2, 3, 4]
```



&emsp;&emsp;==不过==  ES6 中的[展开语法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_syntax) 可以更加简洁的实现这些功能：

```js
let arr1 = [1, 2], arr2 = [3, 4];

arr1.push(...arr2); // arr1 : [1, 2, 3, 4]

Math.max(...arr1); // 4
```



## bind 函数

[MDN 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)

`Function.prototype.bind(obj, arg1, arg2, ...)`

&emsp;&emsp;简单来说，`bind` 主要用于绑定一个函数的 `this` 指向。



### 例1

&emsp;&emsp;由于 Javascript 的变量域问题，有时会使用 `that = this` ：

```js
var myObj = {
    func1: function () { ... },
    
    func2: function () { ... },

    getAsyncData: function (cb) {
        cb();
    },

    render: function () {
        var that = this;
        this.getAsyncData(function () {
            that.func1();
            that.func2();
        });
    }
};

myObj.render();
```

&emsp;&emsp;为了不改变 `myobj` 中的上下文关系，使用了 `that.fun()` 来调用函数。

&emsp;&emsp;这种方式可以使用 `Function.protype.bind()` 来简化：

```js
render: function () {
    this.getAsyncData(function () {
        this.func1();
        this.func2();
    }.bind(this));
}
```



&emsp;&emsp;关于 `bind()` 函数的内部逻辑可以用一个简单的小例子来解释：

```js
Function.prototype.bind = function (scope) {
    var fn = this;
    return function () {
        return fn.apply(scope);
    };
}
var foo = {
    x: 3
}

var x = 10;

var bar = function(){
    console.log(this.x);
}
bar(); // 10 , 此时 this 指向全局作用域

var boundFunc = bar.bind(foo);
boundFunc(); // 3
```
















