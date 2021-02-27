---
title: webpack 配置 babel
index_img: 'https://gitee.com/ylea/imagehost/raw/master/index_img/webpack.png'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/webpack.png'
date: 2020-09-19 20:41:48
categories:
    - 工具
    - webpack
tags:
    - webpack
    - babel
---



&emsp;&emsp;根据 `webpack` 版本的不同，安装的 `babel` 依赖有些区别，下面的是 **webpakc 4.2 及以上版本**安装的依赖。 `4.2` 之前的是：`babel-core`、`babel-preset-env`



&emsp;&emsp;要配置 `babel` 首先需要安装 `@babel/core`，这是 `babel` 的核心编译包：

`cnpm install --save-dev @babel.core`



&emsp;&emsp;然后需要安装 `babel-loader` 来帮助我们加载使用 `babel`：

`cnpm i --save-dev babel-loader`



&emsp;&emsp;这时候，在 `webpack.config.js` 中 `rules` 的代码是这样的：

```js
module:{
    rules: [
        {
            test: /\.js$/,
            use: {
                loader: 'babel-loader'
            },
            exclude:path.resolve(__dirname,'/node_modules'),
            include: path.join(__dirname, 'src_1')
        }
    ]
}
```

&emsp;&emsp;上面的代码中， `exclude` 表示不想被 `babel` 处理的文件目录，`include` 表示需要被处理的目录，不加 `include` 的话应该就是会处理除 `exclude` 以外的其他目录。



&emsp;&emsp;然后，我们可以选择 `babel` 的解析标准，比如我们需要按照 ES6 的标准来，那么就需要装一个 `babel-preset-es2015`，若要按照  ES7 的标准，就需要按照 `babel-preset-es2016`。不过我们可以直接安装一个 `@babel/preset-env` ，它等价于 `babel-preset-latest`，可以编译所有新规范的代码：

`cnpm install --save-dev @babel/preset-env`

&emsp;&emsp;这下，我们的 webpack 中是这样的：

```js
module:{
    rules: [
        {
            test: /\.js$/,
            use: {
                loader: 'babel-loader',
                options: {
                    presets: ['@babel/preset-env']
                }
            },
            exclude:path.resolve(__dirname,'/node_modules'),
            include: path.join(__dirname, 'src_1')
        }
    ]
}
```

&emsp;&emsp;`presets` 中还有其他选项可以帮助我们兼容 js，可根据自己的需求来，具体可看：[babel-preset-env](https://babeljs.io/docs/en/babel-preset-env) 



&emsp;&emsp;除了 webpack，我们还需要在工程目录下创建一个 `.babelrc` 文件，把关于 babel 的配置放进去：

```json
{
    "presets": ["@babel/preset-env"],
}
```



&emsp;&emsp;一般来说 babel 这样配置就差不多了，不过在 `build` 的时候项目里在类中的箭头函数处报了错，提示还需要一个插件 `@babel/plugin-proposal-class-properties`

&emsp;&emsp;因此按照提示安装：

`cnpm i --save-dev @babel/plugin-proposal-class-properties`



&emsp;&emsp;最后， `package.json` 就是这样的：

```json
"devDependencies": {
    "@babel/core": "^7.12.10",
    "@babel/plugin-proposal-class-properties": "^7.12.1",
    "@babel/preset-env": "^7.12.11",
    "babel-loader": "^8.2.2"
}
```



&emsp;&emsp;然后还需要修改 `.babelrc` ：

```json
{
    "presets": ["@babel/preset-env"],
    "plugins": ["@babel/plugin-proposal-class-properties"]
}
```