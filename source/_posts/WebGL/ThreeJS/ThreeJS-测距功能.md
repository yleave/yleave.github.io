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



测距功能，也就是选择两点，计算它们的距离，实现效果大致如下：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost@master/img/ranging.gif" alt="ranging" style="zoom:80%;" />

上图中主要涉及几个操作：

- 点击鼠标左键选点，点击鼠标右键停止选点，若选择点数超过两点，则两点绘制一条线段
- 动态绘制线段
- 动态绘制距离
- 确定两点后将距离文字居中
- 按下 ESC 键撤销上一步操作



## 选点绘线

首先，我们需要通过鼠标在三维空间中选点，但是我们的屏幕是二维的，还有一维不知道，因此没办法直接凭空选点，因此目前的选点都是基于某个物体来的，即在物体上选点。

那么要如何获取鼠标点击的位置呢，首先需要知道[标准设备坐标转空间坐标](https://yleave.top/2020/10/17/WebGL/ThreeJS/ThreeJS-%E5%B1%8F%E5%B9%95%E5%9D%90%E6%A0%87%E4%B8%8E%E4%B8%96%E7%95%8C%E5%9D%90%E6%A0%87%E4%BA%92%E8%BD%AC/)，然后使用 [Raycaster](https://threejs.org/docs/index.html#api/zh/core/Raycaster) 根据这个点从相机位置发出一条射线，并检测与这条射线相交的物体，即可获得鼠标点击的位置：

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

选了点后，需要在该点位置绘制一个圆点表示，这边使用了[SphereGeometry](https://threejs.org/docs/index.html#api/zh/geometries/SphereGeometry：) 来绘制圆点：

```js
createPoint = (pos) => {
    var mat = new THREE.MeshBasicMaterial({color: 0xff0000});
    var sphereGeometry = new THREE.SphereGeometry(0.3, 32, 32);
    var sphere = new THREE.Mesh(sphereGeometry, mat);
    sphere.position.set(pos.x, pos.y, pos.z);
    return sphere;
};
```

两点之间绘制一条线段：

```js
createLine = (p1, p2) => {
    let lineGeometry = new THREE.Geometry();
    let lineMaterial = new THREE.LineBasicMaterial({ color: 0x00ff00 });

    lineGeometry.vertices.push(p1, p2);
    let line = new THREE.Line(lineGeometry, lineMaterial);
    return line;
};
```

## 动态绘制线段和显示距离

在选完起始点后，鼠标移动过程中需要动态绘制线段并显示当前位置和起始点的距离，距离计算使用 `Vector3.distanceTo(vec)` 即可，而动态效果则是不断重复的往场景中添加新的线段和文字移除旧的线段和文字。

拆分一下，绘制线段已经有了，现在是创建 `label` 显示距离，`label` 的创建需要先使用 [FontLoader](https://threejs.org/docs/index.html#api/zh/loaders/FontLoader) 来制指定字体，再使用 [TextBufferGeometry](https://threejs.org/docs/index.html#api/zh/geometries/TextGeometry) 来生成标签，下面是一些关键代码：

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

在鼠标移动过程中绘制：

```js
const p1 = points[points.length-1];
const mouse = new THREE.Vector2(
    (e.clientX / width) * 2 - 1, 
    -(e.clientY / height) * 2 + 1
);

let obj = this.getIntersects(mouse);

if (scene.getObjectByName('active-line')) {
    scene.remove(scene.getObjectByName('active-line'));
}

if (scene.getObjectByName('active-label')) {
    scene.remove(scene.getObjectByName('active-label'));
}

if (obj) {
    let line = this.createLine(p1.position, obj.point);
    line.name = 'active-line';
    scene.add(line);

    let dis = p1.position.distanceTo(obj.point).toFixed(2);
    let label = this.createLabel(dis + '', obj.point);
    label.name = 'active-label';
    scene.add(label);
}
```



## 文字居中

知道了如何创建标签后，文字居中其实就很简单了，根据两个点的位置计算他们的中点，再在中点处添加标签即可：

```js
let start = points[points.length-1];
let line = this.createLine(start.position, p.position);
let dis = start.position.distanceTo(p.position).toFixed(2);
let pos = new THREE.Vector3().copy(start.position);
pos.add(p.position).multiplyScalar(0.5);
let label = this.createLabel(dis + '', pos);

scene.add(line);
scene.add(label);
```

## 撤销操作

使用 `keydown` 对键盘事件进行监听，当 `event.key === Escape` 时我们就撤销前面绘制的点和线和距离 label （如果有这些对象的话），也就是我们之前要使用一个数据结构，比如数组来保存我们添加到场景中的这些对象，然后根据添加顺序再一个个移除即可，因为主要就使用了 `scene.remove` 和顺序逻辑判断，这边就不贴代码了。

