---
title: ThreeJS 杂记
index_img: https://gitee.com/ylea/imagehost/raw/master/img/three.jpg
banner_img: https://gitee.com/ylea/imagehost/raw/master/banner_img/24.jpg
date: 2021-01-08 16:20:27
categories:
    - WebGL
    - ThreeJS
tags:
    - 矩阵
---



# ThreeJS 中的矩阵存储

[ThreeJS 文档](https://threejs.org/docs/index.html#api/zh/math/Matrix4)



&emsp;&emsp;在 ThreeJS 中，矩阵是以**列主序**的方式存储的，也就是说有**行主序**矩阵：
$$
\begin{bmatrix}  \color{red}1 & \color{red}2 & \color{red}3 & \color{red}{4} \\ \color{green}5 & \color{green}6 & \color{green}7 & \color{green}{8} \\ \color{blue}9 & \color{blue}10 & \color{blue}11 & \color{blue}{12} \\ \color{purple}13 & \color{purple}14 & \color{purple}15 & \color{purple}16 \end{bmatrix}
$$


&emsp;&emsp;其在 ThreeJS 中的存储就变成了：
$$
\begin{bmatrix}  \color{red}1 & \color{green}5 & \color{blue}9 & \color{purple}{13} \\ \color{red}2 & \color{green}6 & \color{blue}10 & \color{purple}{14} \\ \color{red}3 & \color{green}7 & \color{blue}11 & \color{purple}{15} \\ \color{red}4 & \color{green}8 & \color{blue}12 & \color{purple}16 \end{bmatrix}
$$
&emsp;&emsp;使用代码测试：

```js
var A = new THREE.Matrix4();
A.set(1, 2, 3, 4,
      5, 6, 7, 8,
      9, 10, 11, 12,
      13, 14, 15, 16);
console.log('A:',A);

var B = new THREE.Matrix4();
B.set(16, 15, 14, 13,
      12, 11, 10, 9,
      8, 7, 6, 5,
      4, 3, 2, 1);
console.log('B:',B);

var C = new THREE.Matrix4();
C.multiplyMatrices (A, B);    
console.log('A · B:',C);
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210118163104957.png" alt="image-20210118163104957" style="zoom:80%;" />

&emsp;&emsp;在网上找一个在线矩阵计算器，相对应的计算结果如下：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/oepd9g9ip6.jpeg" alt="oepd9g9ip6" style="zoom:80%;" />

# ThreeJS 前乘与后乘 

&emsp;&emsp;在 ThreeJS 中，**后乘**的方法是 `multiply()` ，也就是 `A.multiply(B) = A * B` ，结果会保存在 `A` 中：

```js
var A = new THREE.Matrix4();
A.set(1, 2, 3, 4,
      5, 6, 7, 8,
      9, 10, 11, 12,
      13, 14, 15, 16);

var B = new THREE.Matrix4();
B.set(16, 15, 14, 13,
      12, 11, 10, 9,
      8, 7, 6, 5,
      4, 3, 2, 1);

A.multiply(B);
console.log('A:',A);
console.log('B:',B);
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210118163927855.png" alt="image-20210118163927855" style="zoom:80%;" />

&emsp;&emsp;**前乘**方法是：`premultiply()`，即 `A.premultiply(B) = B * A`，同样结果保存在 `A` 中：

```js
var A = new THREE.Matrix4();
A.set(1, 2, 3, 4,
      5, 6, 7, 8,
      9, 10, 11, 12,
      13, 14, 15, 16);

var B = new THREE.Matrix4();
B.set(16, 15, 14, 13,
      12, 11, 10, 9,
      8, 7, 6, 5,
      4, 3, 2, 1);

A.premultiply(B);
console.log('A:',A);
console.log('B:',B);
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210118164228142.png" alt="image-20210118164228142" style="zoom:80%;" />

<img src="https://gitee.com/ylea/imagehost/raw/master/img/n3nfz0wsrn.jpeg" alt="n3nfz0wsrn" style="zoom: 80%;" />

---

REF ：https://cloud.tencent.com/developer/article/1694841

