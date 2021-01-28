---
title: react简例
index_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/index_img/眼镜.jpg'
banner_img: 'https://cdn.jsdelivr.net/gh/yleave/imagehost/banner_img/44.png'
date: 2021-01-23 17:06:09
categories:
tags:
---



# 根据给定的 schema 实现一个简单的渲染引擎

有 `schema.js` 如下，`schema` 中记录了组件的类型、属性和子组件：

```js
export default {
    componentName: 'Form',
    props: {
        labelCol: { span: 8 },
        wrapperCol: { span: 8 },
        name: 'basic',
        initialValues: { remember: true }
    },
    children: [{
        componentName: 'Form.Item',
        props: {
            label: '用户名',
            name: 'username',
            rules: [{ required: true, message: '请输入你的用户名!' }]
        },
        children: [{
            componentName: 'Input'
        }]
    }, {
        componentName: 'Form.Item',
        props: {
            label: '密码',
            name: 'password',
            rules: [{ required: true, message: '请输入你的密码!' }]
        },
        children: [{
            componentName: 'Input.Password'
        }]
    }, {
        componentName: 'Form.Item',
        props: {
            wrapperCol: { offset: 8, span: 8 },
            name:'remember',
            valuePropName: 'checked'
        },
        children: [{
            componentName: 'Checkbox',
            children: ['Remember me']
        }]
    }, {
        componentName: 'Form.Item',
        props: {
            wrapperCol: { offset: 8, span: 8 },
        },
        children: [{
            componentName: 'Button',
            props: {
                type: 'primary',
                htmlType: 'submit'
            },
            children: ['提交']
        }]
    }]
}
```

根据这个 `schema` 来创建组件渲染页面。



渲染引擎主要使用 `React.createElement` 来创建组件

`createElement` API 的相关文档有：

https://zh-hans.reactjs.org/docs/react-api.html#createelement

https://zh-hans.reactjs.org/docs/jsx-in-depth.html



## 简单示范

首先有一个 `index.js` 页面渲染 `App` 组件：

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

我们在 `App.js` 中使用 `createElement` 来创建组件：

```js
import React, { Component } from 'react';
import { Button } from 'antd';

// 使用原生 DOM
var child1 = React.createElement('li', null, 'First Text Content');
var child2 = React.createElement('li', null, 'Second Text Content');
var child3 = React.createElement('li', null, 'Third Text Content');
var App = React.createElement('ul', { className: 'my-list' }, [child1, child2, child3]);

// 使用 antd 组件
const App = React.createElement(Button, null, '提交');

export default function() {
    return App;
};
```



## 渲染引擎简单实现

根据 `schema`，使用每个组件的对应参数来创建组件：

```js
// RenderEngine.js
import React, { Component } from 'react';

export default class RenderEngine {
    constructor(schema, components) {
        this.schema = schema;
        this.components = components;
    }

    parseSchema = (objs) => {
        const { componentName, props } = objs;
        let children = [];
        const components = this.components;

        if (objs.children) {
            for (let child of objs.children) {
                if (child.componentName) {
                    children.push(this.parseSchema(child));
                } else {
                    children.push(child)
                }
            }
        }
        return React.createElement(
            components[componentName],
            props,
            ...children
        );
    };

    createApp = () => {
        return this.parseSchema(this.schema);
    };
}
```

然后就可以在 `App` 页面调用 `createApp` 来完成 `schema` 中指定的组件创建：

```js
import schema from './schema';
import RenderEngine from './RenderEngine';
import React, { Component } from 'react';
import { Button, Input, Checkbox, Form } from 'antd';
const PassWord = Input.Password;
const Item = Form.Item;

const components = { 
    'Button': Button, 
    'Input': Input, 
    'Form.Item': Item, 
    'Checkbox': Checkbox, 
    'Input.Password': PassWord, 
    'Form': Form 
};

// 渲染引擎的输入为页面描述 schema 和组件依赖的映射 components
const re = new RenderEngine(schema, components);
const App = re.createApp();
export default function() {
    return App;
};
```

































