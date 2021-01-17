---
title: threejs画面拖动事件判断
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/img/three.jpg
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/24.jpg'
date: 2021-01-13 20:42:53
categories:
    - WebGL
    - ThreeJS
tags:
    - 鼠标事件
    - 拖拽判断
---



&emsp;&emsp;前因：想实现一个小功能，有一个参数 `lockTiles`，当鼠标在屏幕上拖动时，参数 `lockTiles` 设置为 `true`，停止拖动时，`lockTiles` 重设为 `false`。



&emsp;&emsp;思考了一下，这个功能并不难，有两个方向可以实现这个功能：

- 根据相机是否移动来设置
- 设置鼠标监听事件，使用 `mousedown`、`mousemove` 和 `mouseup` 来判断是否进行了拖动

&emsp;&emsp;不过**在对鼠标进行事件监听时遇到了一些坑点**。。

# 1. 根据相机是否移动来判断是否进行了拖拽

&emsp;&emsp;查阅了 ThreeJS 文档，发现在[轨道控制器 OrbitControls](https://threejs.org/docs/#examples/zh/controls/OrbitControls) 中有几个事件可以用于判断相机是否进行了移动：

- `change`：当相机位置发生改变时触发
- `start`：在对相机进行交互的开始时触发
- `end`：在停止对相机进行交互时触发

&emsp;&emsp;基于我们的需求，这边使用的是 `start` 和 `end` 事件。

&emsp;&emsp;代码很简单：

```js
controls = new OrbitControls( camera, renderer.domElement );

controls.addEventListener('start', lock);
controls.addEventListener('end', unlock);

function lock(e) {
    if (!param.lockTiles) {
        console.log('lock')
        param.lockTiles = true;
    }
}

function unlock(e) {
    if (param.lockTiles) {
        console.log('unlock')
        param.lockTiles = false;
    }
}
```

&emsp;&emsp;不过，对于这种实现方式，当我们**使用滚轮拉近和拉远相机时也会触发** `start` 和 `end` 事件，这个不是我们想要的效果：（`lockTiles` 的设置会反映到右上角的勾选框中，不过在滚轮滚动时，因为触发的太快因此看上去都是一直未被勾选的）

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/flash1.gif" alt="flash1" style="zoom:80%;" />

# 2.设置鼠标监听事件

&emsp;&emsp;拖拽操作可以分解为几个步骤：

1. 鼠标左键按下
2. 鼠标进行拖动
3. 鼠标左键抬起

&emsp;&emsp;因此我们完全可以监听鼠标的动作来判断是否进行了拖拽操作。



&emsp;&emsp;具体是在**鼠标按下**时设置一个时间戳，当**鼠标移动**时，判断当前时间减去之前设置的时间戳是否大于某个阈值，若大于，我们认为此时进行了画面拖拽操作，也就是根据时间差来判断。



&emsp;&emsp;在 threejs 中，我们的画布 `canvas` 就是 `renderer.domElement`，对其设置事件监听即可。



&emsp;&emsp;这时候遇到了坑点，正常来说 `canvas` 元素是支持鼠标事件 `mousedown` 和 `mouseup` 的，不过在实际测试时，发现 `mousedown` 和 `mouseup` 并没有工作，只有 `mousemove` 起效果了。

&emsp;&emsp;最后，经过多次尝试和google，发现可以使用 `pointerdown` 和 `pointerup` 事件来替代。（[pointer event](https://developer.mozilla.org/zh-CN/docs/Web/API/Pointer_events)）

> [`PointerEvent`](https://developer.mozilla.org/zh-CN/docs/Web/API/PointerEvent) 接口继承了所有 [`MouseEvent`](https://developer.mozilla.org/zh-CN/docs/Web/API/MouseEvent) 中的属性，以保障原有为鼠标事件所开发的内容能更加有效的迁移到指针事件。

&emsp;&emsp;然后，基于 `pointer` 事件，写了一个简单的拖拽触发判断类：

```js
export default class MouseDragCheck {
    constructor(props) {
        this.dom = props.dom;
        this.downCb = props.downCb; // 几个事件触发时的回调函数
        this.upCb = props.upCb;
        this.moveCb = props.moveCb;

        this.startClickDown = -1;	// 鼠标按下的时间戳
        this.dragInterval = 100;	// 鼠标拖动的毫秒间隔，大于 100ms 认为它在拖动
    }

    addEventListeners = () => {
        const dom = this.dom;

        dom.addEventListener('pointerdown', this.mouseDown, false);
        dom.addEventListener('pointermove', this.mouseMove, false);
        dom.addEventListener('pointerup', this.mouseUp, false);
    };

    removeEventListeners = () => {
        const dom = this.dom;

        dom.removeEventListener('pointerdown', this.mouseDown, false);
        dom.removeEventListener('pointermove', this.mouseMove, false);
        dom.removeEventListener('pointerup', this.mouseUp, false);
    };

    mouseDown = (e) => {
        this.startClickDown = new Date().getTime();
        if (this.downCb) {
            this.downCb(e);
        }
    };

    mouseMove = (e) => {
        const cur = new Date().getTime();
        if (this.startClickDown !== -1 && cur - this.startClickDown > this.dragInterval) {
            if (this.moveCb) {
                this.moveCb(e);
            }
        }
    };

    mouseUp = (e) => {
        this.startClickDown = -1;
        if (this.upCb) {
            this.upCb(e);
        }
    };
}
```

&emsp;&emsp;然后调用这个类来完成我们对 `lockTiles` 属性设置的需求：

```js
let dragCheck = new DragCheck({
    dom: renderer.domElement,
    moveCb: lock,
    upCb: unlock
});

... 
if (track) {
    dragCheck.addEventListeners();
} else {
    dragCheck.removeEventListeners();
}
```

&emsp;&emsp;这样实现的话，只有画面在进行拖拽时能触发回调，而滚轮滚动时则不会触发：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/flash2.gif" alt="flash2" style="zoom:80%;" />



# 小结

&emsp;&emsp;在 ThreeJS 中对画面的拖拽判断可以使用 `OrbitControls` 中的事件 `start`、`end` 来判断，也可以使用 DOM 中的 `pointerdown`、`pointermove` 和 `pointerup` 来判断，而使用 `mousedown`、`mouseup` 事件是没有效果的。

