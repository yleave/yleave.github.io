---
title: ThreeJS 坐标转换
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/img/three.jpg
banner_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/24.jpg
date: 2020-10-17 12:14:06
categories:
    - WebGL
    - ThreeJS
tags:
    - threeJS
    - 坐标转换
---

# 物体坐标转屏幕坐标

&emsp;&emsp;在 ThreeJS 中，一个物体可看作一个 `Mesh`，`Mesh` 的坐标是用一个 `Vector3` 来表示的，`Vector3` 中包含了 `x`、`y`、`z` 坐标。

## project 方法

&emsp;&emsp;通过 `Vector3 `对象的方法 `project`，方法的参数是相机对象，语句 `worldVector.project(camera);`返回的结果是世界坐标 `worldVector `在 `camera `相机对象矩阵变化下对应的标准设备坐标， 标准设备坐标 `xyz` 的范围是` [-1,1]`。

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20201015163144727.png" alt="image-20201015163144727" style="zoom:80%;" />

&emsp;&emsp;Three 的场景是建立在 `canvas` 之上的，`canvas` 画布的宽高是：`canvas.offsetWidth` 和 `canvas.offsetHeight`，从**屏幕坐标系**来看，画布的中心就是：`(canvas.offsetWidth / 2，canvas.offsetHeight / 2)`，从 WebGL **标准设备坐标系**的角度来看画布中心是坐标原点 `(0, 0)`。

&emsp;&emsp;因此，从标准设备坐标系转换到屏幕坐标系我们可以这样做：

&emsp;&emsp;首先计算出屏幕坐标系中心：

```js
const centerX = canvas.offsetWidth / 2;
const centerY = canvas.offsetHeight / 2;
```

&emsp;&emsp;计算出的 `centerX` 和 `centerY` 同时也表示了坐标轴的一半大小。

&emsp;&emsp;然后，由于**屏幕坐标系的原点是在左上角**，因此转换公式如下：

```js
const standardVec = worldVector.project(camera);

const screenX = Math.round(centerX * standardVec.x + centerX);
const screenY = Math.round(-centerY * standardVec.y + centerY);
```



# Reference

http://www.yanhuangxueyuan.com/Three.js_course/screen.html





