---
title: 'webpack基础1:使用webpack打包js代码'
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/webpack.png'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/webpack.png'
date: 2020-9-14 13:34:05
categories:
    - 工具
    - webpack
tags:
    - webpack
---





[官网](https://webpack.docschina.org/)



&emsp;&emsp;**介绍**：webpack 是一个现代 JavaScript 应用程序的静态模块打包器(module bundler)。当 webpack 处理应用程序时，它会递归地构建一个依赖关系图(dependency graph)，其中包含应用程序需要的每个模块，然后将所有这些模块打包成一个或多个 bundle。

&emsp;&emsp;（简单来说就是在使用ES6的模块编程时，`webpack`会分析这些文件的关系，然后打包成一个 JS 文件，最后直接使用这个 JS 文件即可。

# 从一个示例开始

&emsp;&emsp;一个简单的项目，目录如下：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200615191631329.png" alt="image-20200615191631329" style="zoom:80%;" /> 

`index.html` :

```html
<script src="./src/main.js"></script>
```

`hello.js`:

```js
export default function() {
    console.log('hello webpack');
}
```

`main.js`：

```js
import hello from './hello';

hello();
```

&emsp;&emsp;就这样运行 HTML 页面，会出错：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200615191853433.png" alt="image-20200615191853433" style="zoom:80%;" /> 



&emsp;&emsp;如果使用 `webpack` 来解决，有两种方式：一种是使用配置文件，另一种不使用配置文件。



## 全局安装方式

`webpack`安装：

`npm install -g webpack webpack-cli`



&emsp;&emsp;先看**不使用配置文件的方式：**

&emsp;&emsp;命令行输入的格式如下：`webpack <entry> [<entry>] -o <output>`

&emsp;&emsp;示例：`webpack src/main.js`

&emsp;&emsp;然后，程序会在 `dist` 目录下生成打包好的入口文件：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200615213634589.png" alt="image-20200615213634589" style="zoom:80%;" />

&emsp;&emsp;接着，只要在 `index.html` 中替换 JS 入口文件即可：

```html
<script src="./dist/main.js"></script>
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-202006152139244.png" alt="image-202006152139244" style="zoom:80%;" />

&emsp;&emsp;`main.js` 中生成的打包文件如下：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-2020061521373750.png" alt="image-2020061521373750" style="zoom:80%;" />

&emsp;&emsp;**再看看使用配置文件的方式：**

&emsp;&emsp;在项目中新建一个 `webpack.config.js` 文件，内容为：

```js
const path = require('path');

module.exports = {
  entry: './src/main.js', // 文件入口
  mode: 'development', // 开发环境
  output: {	
    path: path.resolve(__dirname, 'dist'), // 输出文件目录
    filename: 'bundle.js' // 输出文件名
  }
};
```

&emsp;&emsp;然后在命令行中输入 `webpack` 即可：

 <img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200615214345212.png" alt="image-20200615214345212" style="zoom: 67%;" />

&emsp;&emsp;这时候命令行的输出没有黄色字体了，这是因为我们指定了项目文件为开发环境（development)。而之前的打包文件是因为没指定环境，默认为生产环境（production)。

&emsp;&emsp;这时， `bundle.js` 的内容为：与生产环境的`main.js`有很大区别。

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200615214603773.png" alt="image-20200615214603773" style="zoom:67%;" />



## 局部安装方式

&emsp;&emsp;先卸载全局安装的 `webpack` 和 `webpack-cli` : `npm uninstall -g webpack webpack-cli`

&emsp;&emsp;在项目下新建文件 `package.json` ：

```json
{
    "name": "hello-webpack",
    "version": "1.0.0",
    "dependencies": {}
}
```

&emsp;&emsp;运行命令：`npm install --save webpack webpack-cli`

&emsp;&emsp;安装完后再配置运行命令`build`：

```json
{
    "name": "hello-webpack",
    "version": "1.0.0",
    "scripts": {
        "build": "webpack"
    },
    "dependencies": {
        "webpack": "^4.43.0",
        "webpack-cli": "^3.3.11"
    }
}
```

&emsp;&emsp;这时候，运行 `npm run build` 命令也能执行 `webpack`：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200615220311315.png" alt="image-20200615220311315" style="zoom:67%;" />

