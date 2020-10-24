---
title: threeJS 中数学相关内容
index_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/img/three.jpg
banner_img: https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/24.jpg
date: 2020-10-18 20:09:04
categories:
    - WebGL
    - ThreeJS
tags:
    - 矩阵
    - 向量
    - Matrix
    - Box
---




# Three 中的矩阵关系

REF: https://juejin.im/post/6844903510769664014

---

&emsp;&emsp;我们将相机的位置方向, 相机的类型, 物体的位置和形变能转换为 **矩阵**, 将这些矩阵进行一系列计算后, 最终得到**三维投影矩阵**: `u_matrix`

&emsp;&emsp;基于它, 任意给定三维坐标`[x, y, z]`, 我们都能算出相机视平面上的位置:

&emsp;&emsp;**[x, y] = u_matrix \* [x, y, z, 1]**



## THREE 中的矩阵

&emsp;&emsp;THREE定义了场景(Scene)和相机(Camera)， Scene用来添加管理三维物体, Camera用来控制相机的位置， 角度等。

&emsp;&emsp;THREE中定义了下述三个矩阵：

- 相机投影类型：投影矩阵（`ProjectMatrix`）
- 相机的位置和方向: 视图矩阵 (`CameraMatrixWorldInverse` 或 `ViewMatrix`)
- 物体的位置和形变: 物体位置矩阵(`ObjectWorldMatrix`)

## 三维投影矩阵(u_matrix)计算公式

&emsp;&emsp;三维投影矩阵计算公式如下：

```js
const uMatrix = ProjectMatrix * CameraMatrixWorldInverse * ObjectMatrixWorld
```

### 相机投影矩阵（ProjectMatrix）

&emsp;&emsp;在THREE中，通过用不同的相机类实例化，得到不同类型的相机，例如定义一个透视投影相机：

```js
const camera = new THREE.PerspectiveCamera(fov, aspect, near, far);
```

- 获得 `ProjectionMatrix`

  ```js
  camera.projectionMatrix
  ```

  <img src="three.js数学相关内容.assets/image-20200807170747954.png" alt="image-20200807170747954" style="zoom:80%;" />

### 相机视图矩阵(CameraMatrixWorldInverse)

&emsp;&emsp;有的三维引擎或教程，会把视图矩阵称为`ViewMatrix`（例如webglfundamentals）

&emsp;&emsp;视图矩阵的含义是，固定其他因素，我们改变了相机的位置和角度后，它眼中的世界也会发生变化，这种变化就是视图矩阵。

&emsp;&emsp;前面提到，**相机在三维空间中的位置**是`camera.matrixWorld`，而**它的视图矩阵是相机位置矩阵的逆矩阵**`CameraMatrixWorldInverse`，它也符合了我们的生活经验：

- 固定相机，人向左移动
- 固定人，向右移动相机

&emsp;&emsp;这两种情况在相机眼中是一样的。

<img src="three.js数学相关内容.assets/image-20200807170828153.png" alt="image-20200807170828153" style="zoom:80%;" />

&emsp;&emsp;在THREE中，我们一般通过设置`camera`的position和up，调用`lookAt`来改变相机的视图矩阵

```js
camera.position.set(x, y, z);
camera.up.set(x, y, z);
camera.lookAt(x, y, z);
```

- 获得 `CameraMatrixInverse`

  ```js
  camera.matrixWorldInverse
  ```



> &emsp;&emsp;我们知道，最终的投影是在GLSL顶点着色器中计算的。
>
> &emsp;&emsp;在一次绘制中，`ProjectionMatrix`和`CameraMatrixWorldInverse`一般不会发生变化，而`ObjectMatrixWorld`每个物体都可能不同， 所以为了减少顶点着色器中的计算量，有些三维引擎会在javascript程序中提前计算出`ProjectionMatrix * CameraMatrixWorldInverse`的值传递给顶点着色器，这个矩阵一般称为`ViewProjectionMatrix`

### 物体位置矩阵(ObjectWorldMatrix)

&emsp;&emsp;`ObjectWorldMatrix` 描述了物体在三维场景中的位置。

- 获得 `ObjectWorldMatrix`

  ```js
  object.matrixWorld
  ```

&emsp;&emsp;前面提到，THREE中的物体是有层级关系的，所以THREE中物体的`matrixWorld`是通过`local matrix`（`object.matrix`）与父亲的`matrixWorld`递归相乘得到的， 其中的原理可以查阅webglfundamentals中的[这篇教程](https://webglfundamentals.org/webgl/lessons/zh_cn/webgl-scene-graph.html)









# Box3

REF: https://www.mrguo.link/article?id=12
---

&emsp;&emsp;在线案例点击[three.js Box3](http://three.mrguo.link/box3)。

&emsp;&emsp;Box3 在3D空间中表示一个包围盒。其主要用于表示物体在世界坐标中的边界框。它方便我们判断物体和物体、物体和平面、物体和点的关系等等。 

&emsp;&emsp;构造器参数`Box3( min : Vector3, max : Vector3 )`，其参数为两个三维向量，第一个向量为Box3在3D空间中各个维度的最小值，第二个参数为Box3在3D空间中各个维度的最大值，代码如下。

```js
copyvar box = new THREE.Box3(new THREE.Vector3(-2,-2,-2), new THREE.Vector3(2,2,2));
js
```

&emsp;&emsp;这个box就表示3D空间中中心点在`(0,0,0)`，长宽高为`4`的包围盒。









# Vector3

REF: https://www.mrguo.link/article?id=14

---



## Vector3 的方法

### setFromMatrixScale( m: Matrix4 ): this

&emsp;&emsp;从变换矩阵（transformation matrix）`m`中， 设置该向量为其中与缩放相关的元素。

```js
copyvar vec1 = new THREE.Vector3();
var matrix = new THREE.Matrix4().makeScale(1,2,3);
vec1.setFromMatrixScale(matrix);//返回Vector3 {x: 1, y: 2, z: 3}
```

### applyMatrix4( m: Matrix4 ): this

&emsp;&emsp;将该向量乘以四阶矩阵`m`（第四个维度隐式地为`1`）

```js
copyvar vec1 = new THREE.Vector3(1,0,0);
var matrix = new THREE.Matrix4().makeRotationZ(-Math.PI/6);
vec1.applyMatrix4(matrix);//返回值和上面相同
```





# Matrix3

&emsp;&emsp;three.js文档中以行为主的顺序显示矩阵。 但是，如果您正在阅读源代码，您必须对这里列出的任何矩阵进行转置`transpose`，以理解计算。例如：

```js
var matrix3 = new THREE.Matrix3().set( 1,2,3,4,5,6,7,8,9); 
//而其内部elements则展示为： matrix3.elements = [1,4,7,2,5,8,3,6,9];
```

## Matrix3 的属性

&emsp;&emsp;`elements:Array` ：矩阵列优先column-major列表。

&emsp;&emsp;`isMatrix3:Boolean`：用于判定此对象或者此类的派生对象是否是三维矩阵。默认值为 `true`。



## Matrix3的方法

### set() : Matrix3

&emsp;&emsp;三维矩阵不能在构造函数中直接设置参数值，需要通过`set()`方法设置，`set()`方法参数采用**行优先**row-major， 而它们在内部是用列优先column-major顺序存储在数组当中。

### identity():Matrix3

&emsp;&emsp;将此矩阵重置为3x3单位矩阵。

```js
var matrix = new THREE.Matrix3(); 
matrix.identity();//返回elements: (9) [1, 0, 0, 0, 1, 0, 0, 0, 1]
```

### clone():this

&emsp;&emsp;创建一个新的矩阵，元素 elements 与该矩阵相同。

### copy(m:Matrix3):this

&emsp;&emsp;将矩阵`m`的元素复制到当前矩阵中。

### extractBasis( xAxis: Vector3, yAxis: Vector3, zAxis: Vector3 ): Matrix3

&emsp;&emsp;将此矩阵的基提取到提供的三个轴向量中。

```js
var matrix = new THREE.Matrix3(); 
matrix.set( 1,2,3,4,5,6,7,8,9)； 
var a = new THREE.Vector3(); 
var b = new THREE.Vector3(); 
var c = new THREE.Vector3(); 
matrix.extractBasis(a, b, c); 
console.log(a,b,c);
//返回Vector3 {x: 1, y: 4, z: 7} Vector3 {x: 2, y: 5, z: 8} Vector3 {x: 3, y: 6, z: 9}
```

###  setFromMatrix4( m: Matrix4 ): Matrix3

&emsp;&emsp;将当前矩阵设置为`4X4`矩阵，`m`左上`3X3`，

```js
var matrix = new THREE.Matrix3(); 
var matrix4 = new THREE.Matrix4().makeScale(2,2,2); 
matrix.setFromMatrix4(matrix4)//elements: (9) [2, 0, 0, 0, 2, 0, 0, 0, 2]
```

### multiplyScalar( s: number ): Matrix3

&emsp;&emsp;当前矩阵所有的元素乘以该缩放值`s`

```js
var matrix = new THREE.Matrix3(); 
matrix.set( 1,2,3,4,5,6,7,8,9)； 
matrix.multiplyScalar(2)//elements: (9) [2, 8, 14, 4, 10, 16, 6, 12, 18]
```

### determinant(): number

&emsp;&emsp;计算并返回矩阵的行列式determinant 。

```js
var matrix = new THREE.Matrix3(); 
matrix.set( 1,2,3,4,5,6,7,8,9)； 
matrix.determinant()//返回0
```

### getInverse( matrix: Matrix3, throwOnDegenerate?: boolean ): Matrix3

&emsp;&emsp;求传入矩阵`m`的逆矩阵，使用解析法将该矩阵设置为传递矩阵`m`的逆矩阵。

&emsp;&emsp;行列式为零的矩阵不能求逆。如果尝试这样做，该方法将返回一个零矩阵。

```js
var matrix1 = new THREE.Matrix3(); 
var matrix2 = new THREE.Matrix3(); 
matrix1.set(1,2,3,4,5,6,7,8,9)； matrix2.set(1,1,0,0,1,1,0,0,1)； 
new THREE.Matrix3().getInverse(matrix1)//因为行列式的值为0，所以返回零矩阵elements: (9) [0, 0, 0, 0, 0, 0, 0, 0, 0] 
new THREE.Matrix3().getInverse(matrix2)//因为行列式的值不为0，所以返回elements: (9) [1, 0, 0, -1, 1, 0, 1, -1, 1]
```

&emsp;&emsp;求逆矩阵的方法有很多，可以通过伴随矩阵求逆矩阵，也可以行做差来构造逆矩阵。

### transpose(): Matrix3

&emsp;&emsp;将该矩阵转置。

```js
var matrix = new THREE.Matrix3(); 
matrix.set(1,1,0,0,1,1,0,0,1)；//返回elements: (9) [1, 0, 0, 1, 1, 0, 0, 1, 1]; matrix.transpose(); 
console.log(matrix);//返回elements: (9) [1, 1, 0, 0, 1, 1, 0, 0, 1];
```

### getNormalMatrix( matrix4: Matrix4 ): Matrix3

&emsp;&emsp;将这个矩阵设置为给定矩阵的正规矩阵normal matrix（左上角的3x3）。 

&emsp;&emsp;正规矩阵是矩阵`m`的逆矩阵`inverse `的转置`transpose`。

```js
this.setFromMatrix4( matrix4 ).getInverse( this ).transpose();
```

&emsp;&emsp;上面是three.js的源码，从源码可以看出这是setFromMatrix4，getInverse和transpose的组合用法。

```js
var matrix = new THREE.Matrix3(); 
var matrix4 = new THREE.Matrix4().makeScale(2,2,2); 
matrix.getNormalMatrix(matrix4);//返回elements: (9) [0.5, 0, 0, 0, 0.5, 0, 0, 0, 0.5]
```

### transposeIntoArray( r: number[] ): Matrix3

&emsp;&emsp;将当前矩阵的转置`Transposes`存入给定的数组`array : Array`但不改变当前矩阵， 并返回当前矩阵。

```js
var matrix = new THREE.Matrix3(); 
matrix.set(1,1,0,0,1,1,0,0,1)； 
var array = []; 
matrix.transposeIntoArray(array); 
console.log(array);//返回[1, 1, 0, 0, 1, 1, 0, 0, 1]
```

### setUvTransform( tx: number, ty: number, sx: number, sy: number, rotation: number, cx: number, cy: number ): Matrix3

`tx` - x偏移量

`ty` - y偏移量

`sx` - x方向的重复比例

`sy` - y方向的重复比例

`rotation `- 旋转（弧度）

`cx `- 旋转中心x

`cy `- 旋转中心y

&emsp;&emsp;使用偏移，重复，旋转和中心点位置设置UV变换矩阵。

```js
var matrix = new THREE.Matrix3(); 
matrix.setUvTransform(0,0,0,0,0,0.5,0.5); 
console.log(matrix);//返回elements: (9) [0, -0, 0, 0, 0, 0, 0.5, 0.5, 1]
```

### equals( matrix: Matrix3 ): boolean

&emsp;&emsp;如果矩阵`m `与当前矩阵所有对应元素相同则返回`true`。

### fromArray( array: number[], offset?: number ): Matrix3

&emsp;&emsp;使用基于列优先格式column-major的数组来设置该矩阵。

```js
var matrix = new THREE.Matrix3(); 
matrix.fromArray([1,4,7,2,5,8,3,6,9]);//因为是基于列优先原则，所以设置的数组和elements属性相同，不需转置。
```

### toArray( array?: number[], offset?: number ): number[]

&emsp;&emsp;使用列优先column-major格式将此矩阵的元素写入数组中。这是`fromArray`的逆运算。

```js
var matrix = new THREE.Matrix3(); 
matrix.set(1,2,3,4,5,6,7,8,9); 
matrix.toArray(array); 
console.log(array);//返回[1, 4, 7, 2, 5, 8, 3, 6, 9]
```

### multiply( m: Matrix3 ): Matrix3

&emsp;&emsp;将当前矩阵乘以矩阵`m`。

```js
var matrix1 = new THREE.Matrix3(); 
var matrix2 = new THREE.Matrix3(); 
matrix1.set(1,2,3,4,5,6,7,8,9); 
matrix2.setFromMatrix4(new THREE.Matrix4().makeScale(0.5, 0.5, 0.5));//通过4维矩阵，得到三维矩阵 
matrix1.multiply(matrix2);//返回elements: (9) [0.5, 2, 3.5, 1, 2.5, 4, 1.5, 3, 4.5]
```

### premultiply( m: Matrix3 ): Matrix3

&emsp;&emsp;将矩阵`m`乘以当前矩阵。和上面相比这是左乘和右乘的问题。

### multiplyMatrices( a: Matrix3, b: Matrix3 ): Matrix3

&emsp;&emsp;设置当前矩阵为矩阵a x 矩阵b。和上面的方法差不多

### multiplyVector3( vector: Vector3 ): any

&emsp;&emsp;和Vector3.applyMatrix3一样

```js
var matrix1 = new THREE.Matrix3(); 
matrix1.set(1,2,3,4,5,6,7,8,9); 
var vector3 = new THREE.Vector3(1,2,3); 
matrix1.multiplyVector3(vector3);//返回Vector3 {x: 14, y: 32, z: 50}
```







# 分离轴定理（Separating Axis Theorem）

**原文**：https://aotu.io/notes/2017/02/16/2d-collision-detection/

&emsp;&emsp;**概念**：通过判断任意两个 `凸多边形` 在任意角度下的投影是否均存在重叠，来判断是否发生碰撞。若在某一角度光源下，两物体的投影存在间隙，则为不碰撞，否则为发生碰撞。

**图例：**

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/83792-20170706121724065-2000235824.png" alt="83792-20170706121724065-2000235824" style="zoom:80%;" />

&emsp;&emsp;在程序中，遍历所有角度是不现实的。那如何确定 `投影轴` 呢？其实只需对多边形的每一条边都判断一遍即可。且在判断过程中，若提前发现间隙，则不用进行剩下的判断了。

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/83792-20170706121734284-438000124.png" alt="83792-20170706121734284-438000124" style="zoom:80%;" />

&emsp;&emsp;每个投影轴其实就是对应边的法向量：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/image-20200808212950411.png" alt="image-20200808212950411" style="zoom: 50%;" />

**圆形与多边形之间的碰撞检测**

&emsp;&emsp;由于圆形可近似地看成一个有无数条边的正多边形，而我们不可能按照这些边一一进行投影与测试。我们只需将圆形投射到一条投影轴上即可，这条轴就是圆心与多边形顶点中最近的一点的连线，如图所示：

<img src="https://cdn.jsdelivr.net/gh/yleave/imagehost/img/83792-20170706122010112-1512221308.png" alt="83792-20170706122010112-1512221308" style="zoom:80%;" />

&emsp;&emsp;因此，该投影轴和多边形自身的投影轴就组成了一组待检测的投影轴了。另外，圆的投影可以先投影圆心那一点，然后再在两边加上半径即可。



&emsp;&emsp;而对于圆形与圆形之间的碰撞检测依然是最初的两圆心距离是否小于两半径之和。



