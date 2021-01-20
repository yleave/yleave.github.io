---
title: webpack配置导入css
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/webpack.png'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/webpack.png'
date: 2021-01-19 16:38:46
categories:
    - 工具
    - webpack
tags:
    - webpack
---



# 最简单的 CSS 配置

1. 安装依赖：`cnpm i --save-dev css-loader style-loader` 

2. 在 `webpack.config.js` 中添加规则：

   ```js
   module: {
       rules: [
           {
               test: /\.css$/,
               use: [
                   {
                       loader: 'style-loader'  // 可以把css放在页面上
                   },
                   {
                       loader: 'css-loader'    // 放在后面的先被解析
                   }
               ]
           }
   
       ]
   }
   ```

   





































# REF

https://www.jianshu.com/p/a1a4ac781a41