---
title: canvas操作图像
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/img/canvas.jpg'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/img/canvas1.jpg'
date: 2020-09-20 12:12:47
categories:
    - HTML
tags:
    - canvas
    - js截图
---

# canvas 获取截图

&emsp;&emsp;具体是使用 `canvas ` 的 `toDataURL` 方法：

```js
var image = new Image();
image.src = canvas.toDataURL("image/png");
```

&emsp;&emsp;`toDataURL` 的第一个参数是图像类型，默认是 `image/png`，也可设置为 `image/jpeg` 。

&emsp;&emsp;详细可看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement/toDataURL) 的介绍。



&emsp;&emsp;若想将获取的图像下载下来：

```js
var image = new Image();
image.src = canvas.toDataURL("image/png").replace("image/png", "image/octet-stream");
window.location.href= imgUri; // 下载图片
```

# canvas 载入图像

&emsp;&emsp;`canvas` 载入图像有两种方式。一种是使用 `drawImage` 方法载入任何 `canvas` 支持的图像类型，另一种是使用 `putImageData` 方法或是通过循环遍历的方式根据像素值加载图像。

## 使用 drawImage 载入图像

&emsp;&emsp;`canvas` 支持的图像类型有：[CSSImageValue](https://developer.mozilla.org/zh-CN/docs/Web/API/CSSImageValue)，[HTMLImageElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLImageElement)，[SVGImageElement](https://developer.mozilla.org/zh-CN/docs/Web/API/SVGImageElement)，[HTMLVideoElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLVideoElement)，[HTMLCanvasElement](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLCanvasElement)，[ImageBitmap](https://developer.mozilla.org/zh-CN/docs/Web/API/ImageBitmap) 或者[OffscreenCanvas](https://developer.mozilla.org/zh-CN/docs/Web/API/OffscreenCanvas)。

&emsp;&emsp;以最常见的 `HTMLImageElement` 类型的图像为例，也就是使用 [Image](https://developer.mozilla.org/zh-CN/docs/Web/API/HTMLImageElement/Image) 方法或是 `<img>` 标签创建的对象：

```js
var image = new Image();
image.src = 'picture.jpg';

// or 
// <img id="picture" src="..." width="300" height="227">
var image = document.getElementById('picture');
```

&emsp;&emsp;然后，就能使用 `drawImage` 方法载入这个对象：

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");

ctx.drawImage(image, 0, 0);
```

&emsp;&emsp;对于 `new Image` 方式创建的 `image` ，需要等到图片加载完成后才能载入：

```js
var image = new Image();
image.src = 'picture.jpg';

image.onload = function() {
    ctx.drawImage(image, 0, 0);
};
```

&emsp;&emsp;`drawImage` 方法有多个参数，除了第一个 `imageSource` 是必须的，其他参数都是可选的，其他参数设置了源图像和目标图像的起始位置和截取范围：

&emsp;&emsp;`drawImage(image, source_x, source_y, source_width, source_height, x, y, width, heigh)`

&emsp;&emsp;因此，`drawImage` 方法不仅能够普通的载入图像，还能够在载入图像的过程中对图像进行裁剪和缩放。

&emsp;&emsp;具体可参考 [MDN1](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/drawImage) 和 [MDN2](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Using_images)



## 使用 putImageData 载入图像

&emsp;&emsp;首先，使用该方法，我们需要一个 [ImageData](https://developer.mozilla.org/zh-CN/docs/Web/API/ImageData) 的对象。这个对象中存储着 `canvas` 对象真实的像素数据。

`ImageData` 对象包含以下几个**只读**属性：

- `width` : 图片宽度，单位是像素
- `height` : 图片高度，单位是像素
- `data `: [Uint8ClampedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray)类型的一维数组，包含着 **RGBA** 格式的整型数据，范围在0至255之间（包括255）。

&emsp;&emsp;[Uint8ClampedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray) 包含高度 × 宽度 × 4 bytes 数据，索引值从 `0` 到 `(高度 × 宽度 × 4 ) - 1`



&emsp;&emsp;下面将图像从一个 `canvas` 复制到一个新的 `canvas`，涉及两个函数：[getImageData](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/getImageData) 和 [putImageData](https://developer.mozilla.org/zh-CN/docs/Web/API/CanvasRenderingContext2D/putImageData)

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");
var imgData = ctx.getImageData(left, top, width, height);

var newCvs = document.getElementById("new_canvas");
var cewCtx = newCvs.getContext("2d");
cewCtx.putImageData(imgData, 0, 0);
```



&emsp;&emsp;若是已知图像的像素值，也可使用循环遍历的方式来载入图像：

```js
var canvas = document.getElementById("canvas");
var ctx = canvas.getContext("2d");

// 创建一个空的 ImageData 对象
var img = context.createImageData(width, height);
var data = img.data;

// 按 RGBA 格式存储数据
for (let i = 0; i < height; i++) {
    var l = height - 1 - i;
    var s1 = l * width * 4;
    var s2 = i * width * 4;
    for (var j = 0; j < width * 4; j++) {
        data[s1 + j] = imageData[s2 + j]
    }
}

context.putImageData(img, 0, 0);
```



&emsp;&emsp;更多详细内容可参考 [像素操作](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas)