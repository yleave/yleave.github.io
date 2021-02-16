---
title: ThreeJS 测距功能
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/img/three.jpg
banner_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/24.jpg
date: 2021-01-27 23:32:35
categories:
    - WebGL
    - ThreeJS
tags:
    - 测距
    - Three绘制标签
    - Three动态绘制线段
---



&emsp;&emsp;测距功能，也就是选择两点，计算它们的距离，实现效果大致如下：

![](https://img-blog.csdnimg.cn/20210216173934792.gif)


&emsp;&emsp;上图中主要涉及几个操作：

- 点击鼠标左键选点，点击鼠标右键停止选点，若选择点数超过两点，则两点绘制一条线段，两点中心添加一个距离标签
- 动态绘制点和线段
- 动态绘制距离标签
- 确定两点后添加 xyz 辅助线
- 按下 ESC 键撤销上一步操作



## 选点绘线

&emsp;&emsp;首先，我们需要通过鼠标在三维空间中选点，但是我们的屏幕是二维的，还有一维不知道，因此没办法直接凭空选点，因此目前的选点都是基于某个物体来的，即在物体上选点。

&emsp;&emsp;那么要如何获取鼠标点击的位置呢，首先需要知道[标准设备坐标转空间坐标](https://yleave.top/2020/10/17/WebGL/ThreeJS/ThreeJS-%E5%B1%8F%E5%B9%95%E5%9D%90%E6%A0%87%E4%B8%8E%E4%B8%96%E7%95%8C%E5%9D%90%E6%A0%87%E4%BA%92%E8%BD%AC/)，然后使用 [Raycaster](https://threejs.org/docs/index.html#api/zh/core/Raycaster) 根据这个点从相机位置发出一条射线，并检测与这条射线相交的物体，即可获得鼠标点击的位置：

```js
const mouse = new THREE.Vector2();

// 标准设备坐标转空间坐标
mouse.x = (e.clientX / width) * 2 - 1;
mouse.y = -(e.clientY / height) * 2 + 1;

let raycaster = new THREE.Raycaster();
raycaster.setFromCamera(mouse, camera);

let intersects = raycaster.intersectObjects(scene.children);
if(intersects.length > 0) {
    return intersects[0].point;	// point 即点击坐标
}
```

&emsp;&emsp;选了点后，需要在该点位置绘制一个圆点表示，这边使用了[SphereGeometry](https://threejs.org/docs/index.html#api/zh/geometries/SphereGeometry：) 来绘制圆点：

```js
createPoint = (pos, config={color:0x009bea, size:0.3}) => {
    let mat = new THREE.MeshBasicMaterial({color: config.color || 0x009bea});
    let sphereGeometry = new THREE.SphereGeometry(config.size || 0.3, 32, 32);
    let sphere = new THREE.Mesh(sphereGeometry, mat);
    sphere.position.set(pos.x, pos.y, pos.z);
    return sphere;
};
```

&emsp;&emsp;两点之间绘制一条线段：

```js
createLine = (p1, p2, config={color:0x009bea}) => {
    let lineGeometry = new THREE.Geometry();
    let lineMaterial = new THREE.LineBasicMaterial({ color: config.color });

    lineGeometry.vertices.push(new THREE.Vector3().copy(p1), new THREE.Vector3().copy(p2));

    let line = new THREE.Line(lineGeometry, lineMaterial);
    return line;
};
```

## 绘制标签
&emsp;&emsp;绘制标签也就是在屏幕上绘制文字，有两种方法可以生成文字，一种是使用 [TextBufferGeometry](https://threejs.org/docs/index.html#api/zh/geometries/TextGeometry) 来创建 3d 文字，另一种是使用 [CSS2DRenderer](https://threejs.org/docs/index.html#examples/zh/renderers/CSS2DRenderer) 和 `CSS2DObject` 来自定义 2d 标签。
&emsp;&emsp;作为标签，肯定是选择后者更为合适，前者就是单纯的 3d 文字，而后者因为是使用 `div` 元素创建的，因此可以更自由的定义标签的样式、且标签的大小和朝向会跟随相机，并且不会被模型遮挡。

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20210216193101313.png" alt="image-20210216193101313" style="zoom:80%;" />


&emsp;&emsp;不过不同场景实现方案的选择也会不同，因此这边都记录一下。

### 1.使用 TextGeometry 创建标签文字

&emsp;&emsp;`label` 的创建需要先使用 [FontLoader](https://threejs.org/docs/index.html#api/zh/loaders/FontLoader) 来制指定字体，再使用 [TextBufferGeometry](https://threejs.org/docs/index.html#api/zh/geometries/TextGeometry) 来生成标签，下面是一些关键代码：

```js
const loader = new THREE.FontLoader();
loader.load('https://raw.githubusercontent.com/mrdoob/three.js/master/examples/fonts/gentilis_regular.typeface.json', function(font) {
    this.font = font;
}.bind(this));


createLabel = (name, location) => {
    const textGeo = new THREE.TextBufferGeometry(name, {
        font: this.font,
        size: 0.8,
        height: 0.1,
        curveSegments: 1
    });

    const textMaterial = new THREE.MeshBasicMaterial({ color: 0xffffff });
    const textMesh = new THREE.Mesh(textGeo, textMaterial);
    textMesh.position.copy(location);
    // 根据自己的坐标系设置进行旋转将文字水平显示
    textMesh.rotation.x = 0.5 * Math.PI;
    textMesh.rotation.y = Math.PI;

    return textMesh;
};
```

### 2. 使用 CSS2DObject 创建标签

&emsp;&emsp;首先，要让 `CSS2DObject` 创建的标签起效果，需要使用 `CSS2DRenderer` 来渲染，因此场景中的 `renderer` 会有两个，一个 `WebGLRenderer` 用来渲染模型，一个 `CSS2DRenderer` 用来渲染标签，需要注意的是要使用 `css2drenderer` 来创建 `controls` ：

```js
import { CSS2DRenderer, CSS2DObject } from 'three/examples/jsm/renderers/CSS2DRenderer';
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls';

renderer = new WebGLRenderer( { antialias: true } );
renderer.setPixelRatio( window.devicePixelRatio );
renderer.setSize( canvas.width, canvas.height );
canvas.appendChild(renderer.domElement);

labelRenderer = new CSS2DRenderer();
labelRenderer.setSize(canvas.width, canvas.height);
labelRenderer.domElement.style.position = 'absolute';
labelRenderer.domElement.style.top = '0px';
canvas.appendChild(labelRenderer.domElement);

controls = new OrbitControls( camera, labelRenderer.domElement );
```

&emsp;&emsp;然后就可以使用 `CSS2DObject` 来生成标签了：

```js
createLabel = (text, pos) => {
   const div = document.createElement('div');
    div.className = 'label';
    div.textContent = text;
    
    const divLabel = new CSS2DObject(div);
    divLabel.position.set(pos.x, pos.y, pos.z);

    return divLabel;
}
```

&emsp;&emsp;可以根据设置的 `className` 来设置标签样式：

```css
.label {
    margin-top: -1em;
    border: 10px;
    border-radius: 8px;
    width: 85px;
    text-align: center;
    cursor: pointer;
    color: rgb(0, 155, 234);
    line-height: 1.2;
    background-color: rgb(244, 244, 244);
    box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.25);
}

.label:hover {
    box-shadow: 0px 0px 20px rgba(0, 155, 234, 0.8);
}
```

## 动态绘制点、线和标签

&emsp;&emsp;在选点过程中(鼠标移动过程中)，若鼠标在模型之上，鼠标位置处会显示一个圆点，表示此处可点选，在选完一个点后，鼠标移动过程中鼠标位置和选点之间会有连线，且线段的距离会动态变化。

&emsp;&emsp;这些动态效果主要都是通过自定义的名称来查找修改的，即使用了 `scene.getObejctByName()` 来找到这些动态的物体，然后再通过修改其位置和文字信息来产生动态效果（原本是通过 `scene.add` 和 `scene.remove` 来实现，后面做了优化，通过 `geometry.vertices` 和 `position` 来实现），以动态的线段为例：

```js
let activeLine = scene.getObjectByName('active-line');
if (activeLine) {
    activeLine.geometry.vertices[1].set(obj.point.x, obj.point.y, obj.point.z);
    activeLine.geometry.verticesNeedUpdate = true;
} else {
    activeLine = this.createLine(p1.position, obj.point);
    activeLine.name = 'active-line';
    scene.add(activeLine);
}
```
&emsp;&emsp;动态点：

```js
let obj = this.getIntersects(mouse);

let activePoint = scene.getObjectByName('active-point');

// 若点击位置不为空且场景中不存在活动的点，那么创建活动的点
if (obj && !activePoint) {
    activePoint = this.createPoint(obj.point);
    activePoint.material.transparent = true;
    activePoint.material.opacity = 0.6;
    activePoint.name = 'active-point';
    scene.add(activePoint);
} else if (obj) {   // 否则若点击位置不为空且存在活动的点，更新这个点的位置
    activePoint.position.set(obj.point.x, obj.point.y, obj.point.z);
} else if (activePoint) {    // 否则点击位置为空且存在活动的点，清除活动的点
    scene.remove(activePoint);
}
```

&emsp;&emsp;动态标签：

```js
let dis = p1.position.distanceTo(obj.point).toFixed(2);
let pos = new THREE.Vector3().copy(p1.position);
pos.add(obj.point);
pos.multiplyScalar(0.5);

let label = scene.getObjectByName('active-label');
if (label) {
    label.element.textContent = '~' + dis;
    label.position.set(pos.x, pos.y, pos.z);
} else {
    label = this.createLabel('~' + dis, pos);
    label.name = 'active-label';
    scene.add(label);
}
```

## 绘制辅助线
&emsp;&emsp;有了前面绘制线段的方法，绘制辅助线就比较简单了，只要再确定两个中间点，加上线段起始点和终点四个点就能绘制三条线。

&emsp;&emsp;设线段起始点是 `start`，终点是 `p` ，每两点间变化一个坐标轴的值就能达到我们的目的。我们从起始点开始，首先，绘制 `y` 轴的线段，那么就使用起始点 `start` 和新的点`py : Vector3(start.x, p.y, start.z)` 绘制线段，再使用 `py` 和新的中间点 `pz: Vector3(start.x, p.y, p.z)` 来绘制 `z` 轴线段，最后使用 `pz` 和终点 `p` 来绘制 `x` 轴线段。

```js
let py = new THREE.Vector3(start.position.x, p.position.y, start.position.z);
let pz = new THREE.Vector3(start.position.x, p.position.y, p.position.z);

let liney = this.createLine(start.position, py, {color: 0xff0000});
let linez = this.createLine(py, pz, {color: 0x00ff00});
let linex = this.createLine(pz, p.position, {color: 0x0000ff});
```

## 撤销操作

&emsp;&emsp;使用 `keydown` 对键盘事件进行监听，当 `event.key === Escape` 时我们就撤销前面绘制的点和线和距离 label （如果有这些对象的话），也就是我们之前要使用一个数据结构，比如数组来保存我们添加到场景中的这些对象，然后根据添加顺序再一个个移除即可，因为主要就使用了 `scene.remove` 和顺序逻辑判断，这边就不贴代码了。