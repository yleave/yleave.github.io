---
title: ThreeJS快照
index_img: https://gitee.com/ylea/imagehost/raw/master/img/three.jpg
banner_img: https://gitee.com/ylea/imagehost/raw/master/banner_img/24.jpg
date: 2020-09-20 12:13:38
categories:
    - WebGL
    - ThreeJS
tags:
    - threeJS
    - 快照
    - canvas
---

&emsp;&emsp;假设有一个需求，需要获取 Three 场景的一张快照，然后再将其显示在屏幕上。

# 1. 获取快照

&emsp;&emsp;对于这个需求，一种方法是在创建 `WebGLRenderer` 时设置一个 `preserveDrawingBuffer` 参数为 `true`，然后再调用场景中的 `canvas` 的 `toDataURL` 方法来获取某一帧的 `base64` 格式的图像数据：

```js
renderer = new WebGLRenderer({
    preserveDrawingBuffer :true 
});

const image = new Image();
image.src = renderer.domElement.toDataURL();
```

&emsp;&emsp;而 `preserveDrawingBuffer ` 参数的含义是：是否保留缓直到手动清除或被覆盖。若设为 `true`，在任意时刻，前一帧的缓存图像不会被清除，因此可以获取到任意一刻的截图，若未设置的话，获取的截图可能就会是一张空白的图片。不过，这有一个明显的副作用，那就是会更加的消耗性能和占用内存。因此**不推荐**这种方式。

&emsp;&emsp;**另一种方式**就是在获取截图之前先再渲染一下场景，这样就不会因为缓存清空而导致截屏空白了：

```js
renderer.clear();
renderer.render(scene, camera);
const image = new Image();
image.src = canvas.toDataURL("image/png");
```

# 2.在屏幕上显示出来

&emsp;&emsp;既然我们已经能够获取到正确的快照了，那么要将其显示在屏幕上，我们可以再创建一个新的 `canvas`，然后将获取的快照使用这个新的 `canvas` 加载出来就行了，代码如下：

```js
const canvas = renderer.domElement;

renderer.render(scene, camera);

const image = new Image();
const base64 = canvas.toDataURL("image/png");
image.src = base64;

const mycanvas = document.getElementById('myImageEditCanvas');
mycanvas.width = canvas.width;
mycanvas.height = canvas.height;
const ctx = mycanvas.getContext('2d');

image.onload = () => {
    ctx.drawImage(image, 0, 0);
};
```

&emsp;&emsp;使用 `canvas` 载入图像使用了 `drawImage` 方法，它的第二第三个参数的意思是从源图像的哪个位置开始载入，要载入整张图像，填入 `0` 即可。

&emsp;&emsp;`drawImage` 详细一些的介绍可看：[drawImage](https://yleave.top/2020/09/20/HTML/canvas操作图像/)








