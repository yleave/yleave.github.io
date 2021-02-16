---
title: Redux入门学习
index_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/redux1.png'
banner_img: 'https://gitee.com/ylea/imagehost/raw/master/banner_img/redux.png'
date: 2021-01-20 21:02:36
categories:
    - React
tags:
    - redux
    - react-redux
    - redux-thunk
---



参考资源：

[阮一峰博客](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html)

[中文文档](https://www.redux.org.cn/)



Demo 仓库：https://github.com/yleave/redux-demo

# 基本概念

&emsp;&emsp;有几个图片可以帮助理解 `redux`:

&emsp;&emsp;这张图片说明了 `reudx` 的作用：组件间的信息传递不用再只能通过相邻的父子组件进行传递了，而是统一由一个仓库 `store` 来管理这些 `state`：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/%E4%B8%8B%E8%BD%BD.jpg" alt="下载" style="zoom: 150%;" /> 
<img src="https://gitee.com/ylea/imagehost/raw/master/img/OIP%20(1).jpg" alt="OIP (1)" style="zoom: 150%;" />

<img src="https://gitee.com/ylea/imagehost/raw/master/img/OIP.jpg" alt="OIP" style="zoom:150%;" />

 

有两个：

`redux` ： js 的状态管理  如 `createStore`

`react-redux` : 为了在 react 中更容易的使用 ，如 `connect` 、 `provider`



安装：

- `cnpm install --save redux`
- `cnpm i --save react-redux`



&emsp;&emsp;**核心概念：**`redux` 的核心概念就是使用一个个的**可嵌套**的 `reducer` 函数来管理所有的 `state`。

## 三大原则

- **单一数据源**

  整个应用的 [state](https://www.redux.org.cn/docs/Glossary.html#state) 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 [store](https://www.redux.org.cn/docs/Glossary.html#store) 中。

- **`state` 是只读的**

  唯一改变 state 的方法就是触发 [action](https://www.redux.org.cn/docs/Glossary.html#action)，action 是一个用于描述已发生事件的普通对象。

- **使用纯函数来执行修改**

  为了描述 action 如何改变 state tree ，你需要编写 [reducers](https://www.redux.org.cn/docs/Glossary.html#reducer)。

  `Reducer `只是一些纯函数，它接收先前的 state 和 action，并返回新的 state。
  
  

# 使用 Redux 最简单的示例：计数器

&emsp;&emsp;这个示例实现的效果就是：点击 `increasment` 和 `decreasment` 按钮，分别增加、减少界面上显示的数字

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200722213539985.png" alt="image-20200722213539985" style="zoom:80%;" />

**1.** 首先，在 App.js 中先编写一个简单的界面（使用了 bootstrap）：

```jsx
render() {
    return (
        <div className="App">
            <h1 className="jumbotron-heading text-center">0</h1>
            <p className="text-center">
                <button  className='btn btn-primary'>increasment</button>  
                <button  className='btn btn-success'>decreasment</button>
            </p>
        </div>
    );
}
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200722213401691.png" alt="image-20200722213401691" style="zoom:80%;" />

**2.** 在 `src/reducers` 路径下创建文件 `counter.js`：

```js
const counter = ( state = 0, action ) => {
    switch(action.type) {
        case 'INCREASMENT':
            return state + 1;
        case 'DECREASMENT':
            return state - 1;
        default:
            return state;
    }
};

export default counter;
```

**3.** 在 `index.js` 中根据这个 `reducer` 创建 `store`，并将更新 `state` 的方法作为参数传递到 App 组件中，且注册监听器监听 `state` 的状态：

```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore } from 'redux';
import reducer from './reducers/counter';

// 创建 store 仓库
const store = createStore(reducer);
// store.subscribe( () => console.log('state: ', store.getState()));

const render = () => {
    ReactDOM.render(
        <App 
            onIncreasment={ () => store.dispatch({ type: 'INCREASMENT'})}
            onDecreasment={ () => store.dispatch({ type: 'DECREASMENT'})}
            value={ store.getState() }
        />,
        document.getElementById('root')
    );
}; 

// 先运行一次渲染
render();
store.subscribe(render);

```

**4.** 最后，在 App.js 中调用传入的这些参数来更新显示的数字：

```js
import React, { Component } from 'react';
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap';

export default class App extends Component{
    
    render() {
        return (
            <div className="App">
                
                <h1 className="jumbotron-heading text-center">{this.props.value}</h1>
                <p className="text-center">
                    <button onClick={this.props.onIncreasment} className='btn btn-primary'>increasment</button>  
                    <button onClick={this.props.onDecreasment} className='btn btn-success'>decreasment</button>
                </p>
                
            </div>
        );
    }
}
```

## 梳理

**梳理一下 Redux 的使用方法：**

1. 先编写 `reducer` ，用于管理状态，且 `reducer` 是唯一能够修改状态的地方。
2. 使用 `createStore(reducer)`（还有其他参数，不过目前就用到这一个参数）来创建 `store`。
3. 使用 `store.dispatch(action)` 方法来更新 `state`，这个 `action` 可以是多种数据类型，不过本例中是一个对象，然后  `reducer` 中根据 `action.type` 来选择如何更新 `state`。
4. 可使用 `store.getState()` 方法来获取 `state`，这个 `state` 是包含了所有 `state` 的一个对象，本例中只定义了一个数字作为 `state`。
5. 使用 `store.subscribe(listener)` 来注册监听器，监听 `state` 的变化

# 使用 react-redux 的示例：计数器

&emsp;&emsp;使用 `react-redux` 来重写之前的计数器的例子。



**1.** 首先，更改 `index.js` 中的代码，使用 `Provider` 组件来包裹 `App` 组件：

```jsx
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { createStore } from 'redux';
import reducer from './reducers/counter';
import { Provider } from 'react-redux';

// 创建 store 仓库
const store = createStore(reducer);

ReactDOM.render(
    <Provider store={ store }>
        <App />
    </Provider>,
    document.getElementById('root')
);
```

**2.** 更改 `App.js` 中的代码，使用高阶组件 `connect` 来关联 `App` 组件和其状态映射函数：

```jsx
import React, { Component } from 'react';
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap';
import { connect } from 'react-redux';

class App extends Component{
    
    render() {
      console.log('this.props: ', this.props);
      return (
          <div className="App">
              <h1 className="jumbotron-heading text-center">{this.props.counter}</h1>
              <p className="text-center">
                  <button className='btn btn-primary'>increasment</button>  
                  <button className='btn btn-success'>decreasment</button>
              </p>
              
          </div>
      );
  }
}

const mapStateToProps = (state) => {
    return {
        counter: state
    };
};

export default connect(mapStateToProps)(App);
```

&emsp;&emsp;可以观察到页面上输出的 `this.props` 信息，因此可以将 `this.props.counter` 作为显示的数字：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200722222339428.png" alt="image-20200722222339428" style="zoom:80%;" />

**3.** 创建一个 js 文件来封装这个 `reducer` 对应的 `actions`。在 `src/actions/` 下创建文件 `counter.js`：调用这些方法返回对应的 `action`

```js
export function increment() {
    return {
        type: 'INCREMENT'
    };
}

export function decrement() {
    return {
        type: 'DECREMENT'
    };
}
```

**4.** 在 App.js 中定义 `action` 分发到 `props` 的映射，并在按钮中使用这些分发的函数：

```jsx
import React, { Component } from 'react';
import 'bootstrap/dist/css/bootstrap.min.css';
import 'bootstrap/dist/js/bootstrap';
import { connect } from 'react-redux';
import { increment, decrement } from './actions/counter';

class App extends Component{
    
    render() {
      console.log('this.props: ', this.props);
      const { increment, decrement } = this.props;
      return (
          <div className="App">
              <h1 className="jumbotron-heading text-center">{this.props.counter}</h1>
              <p className="text-center">
                  // increment 和 decrement 是 props 中的属性
                  <button onClick={() => { increment(); }} className='btn btn-primary'>increment</button>  
                  <button onClick={() => { decrement(); }} className='btn btn-success'>decrement</button>
              </p>
              
          </div>
      );
  }
}

const mapStateToProps = (state) => {
    return {
        counter: state
    };
};

const mapDispatchToPorps = (dispatch) => {
    return {
        increment: () => { dispatch(increment()) },
        decrement: () => { dispatch(decrement()) }
    };
};
// 将 action 分发的映射也绑定到 App 中，参数的顺序不能改变
export default connect(mapStateToProps, mapDispatchToPorps)(App);
```

&emsp;&emsp;这样，`react-redux` 就自动的帮我们完成 `state` 和 `dispatch ` 到 `App` 组件的绑定，并能够正确的相应 `actions` ，更新 `state`。

&emsp;&emsp;从 打印的 `props` 可以观察到 `state` 的变化：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200722224349311.png" alt="image-20200722224349311" style="zoom:80%;" />

&emsp;&emsp;第 4 步的另外一种写法：（推荐用这种

&emsp;&emsp;使用 `reudx` 的 `bindActionCreators` 方法来将自定义的 `actions` 和 `dispatch` 绑定：

```jsx
import * as counterActions from './actions/counter';
import { bindActionCreators } from 'redux';

const mapDispatchToPorps = (dispatch) => {
    return {
        counterActions: bindActionCreators(counterActions, dispatch)
    };
};

export default connect(mapStateToProps, mapDispatchToPorps)(App);

// render() 中
const { counterActions } = this.props;
return (
    <button onClick={() => { counterActions.increment(); }}>increment</button> 
    <button onClick={() => { counterActions.decrement(); }}>decrement</button>
);
```

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723100226347.png" alt="image-20200723100226347" style="zoom:80%;" />



## 梳理

**`react-redux` 使用流程梳理：**

1. 使用 `Provider` 来包裹需要管理 `state` 的组件，并传入创建好的 `store`：

   ```jsx
   ReactDOM.render(
       <Provider store={ store }>
           <App />
       </Provider>,
       document.getElementById('root')
   );
   ```

2. 创建 `state` 到 组件`props` 的映射，并使用高阶组件 `connect` 将其与组件绑定：

   ```jsx
   const mapStateToProps = (state) => {
       return {
           counter: state
       };
   };
   
   export default connect(mapStateToProps)(App);
   ```

3. 封装`reducer` 中的对应 `actions` ：

   ```js
   export function increment() {
       return {
           type: 'INCREMENT'
       };
   }
   
   export function decrement() {
       return {
           type: 'DECREMENT'
       };
   }
   ```

4. 和 `state` 一样，创建 `actions` 和 `props` 的映射，将其传入 `connect` 中与组件进行绑定：

   ```jsx
   const mapDispatchToPorps = (dispatch) => {
       return {
           increment: () => { dispatch(increment()) },
           decrement: () => { dispatch(decrement()) }
       };
   };
   
   export default connect(mapStateToProps, mapDispatchToPorps)(App);
   ```

   或使用 `redux` 的 `bindActionCreators` 来帮助创建映射，这种方法在很多 `actions` 的时候用起来会比较方便：

   ```jsx
   const mapDispatchToPorps = (dispatch) => {
       return {
           counterActions: bindActionCreators(counterActions, dispatch)
       };
   };
   ```

   

5. `state`和 `actions` 都已经与组件的`props`进行绑定了，因此可以在需要使用 `state` 及更新 `state` 的地方使用 `props` 中的这些属性：

   ```jsx
   const { increment, decrement } = this.props;
   const { counterActions } = this.props;
   return (
       <div className="App">
           <h1>{this.props.counter}</h1>
           <p className="text-center">
               // 未使用 bindActionCreators
               <button onClick={() => { increment(); }} >increment</button>  
               <button onClick={() => { decrement(); }} >decrement</button>
               // 使用 bindActionCreators
               <button onClick={() => { counterActions.increment(); }}>increment</button> 
               <button onClick={() => { counterActions.decrement(); }}>decrement</button>
           </p>
       </div>
   );
   ```




&emsp;&emsp;此外，代码中的 `action` 的 `type` 还可以专门创建一个文件来管理常量：

```js
// src/constants/index.js
// counter
export const INCREMENT = 'INCREMENT';

export const DECREMENT = 'DECREMENT';
```

&emsp;&emsp;这样的话，有两个地方可以修改：

```js
// src/reducers/counter.js
import * as actions from '../constants';

const counter = ( state = 0, action ) => {
    switch(action.type) {
        case actions.INCREMENT:
            return state + 1;
        case actions.DECREMENT:
            return state - 1;
        default:
            return state;
    }
};

export default counter;

// src/actions/counter.js
import * as actions from '../constants';

export function increment() {
    return {
        type: actions.INCREMENT
    };
}

export function decrement() {
    return {
        type: actions.DECREMENT
    };
}
```



&emsp;&emsp;且，页面中增加减少按钮处可设置进行参数传递，来设置一次增加/减少的值：

```js
<button onClick={() => { counterActions.increment(10); }}>increment</button>  
<button onClick={() => { counterActions.decrement(5); }} >decrement</button>
```

1. 先在 `actions` 中接收这个参数，并设置：

```js
export function increment(num) {
    return {
        type: actions.INCREMENT,
        num
    };
}

export function decrement(num) {
    return {
        type: actions.DECREMENT,
        num
    };
}
```

2. 在 `reducer` 中设置每次更新的值：（注意 `actions` 和参数 `action` 的区分

```js
import * as actions from '../constants';

const counter = ( state = 0, action ) => {
    switch(action.type) {
        case actions.INCREMENT:
            return state + action.num;
        case actions.DECREMENT:
            return state - action.num;
        default:
            return state;
    }
};

export default counter;
```

# 使用 combineReducers 合并 reducer

&emsp;&emsp;在应用中，可能会有非常多的数据需要管理，这时就需要多个 `reducer` 来管理，使用 `combineReducers` 来合并这些 `reducers` 可以更加方便的使用这些 `reducers`。



&emsp;&emsp;在页面上再添加一个 `User` 组件，组件中有一个文本标签和一个按钮，目标是点击按钮能后够添加一个字符串并显示在文本中：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723120654431.png" alt="image-20200723120654431" style="zoom:80%;" />

&emsp;&emsp;按照上个例子依葫芦画瓢的添加这个功能：

1. 在 `src/constants/index.js` 中新添加一个常量，作为 `action.type`。

2. 在 `src/actions/` 中新增一个 `action`，`action` 中定义了 `type` 来帮助 `reducer` 中判断执行哪个 操作，也可以定义一些参数在 `reducer` 中使用。

3. 在`src/reducers/` 中新增一个 `reducer` ，来管理这个新功能的 `state`。

4. 在`src/App.js` 中，也就是主组件中的 `mapStateToProps` 和 `mapDispatchToProps` 中添加这个新功能的 `state` 和 `action`

5. 先忽略使用 `reducer` 创建 `store` 这一步，有了之前的步骤后，现在就能在 `render` 中使用 `props` 里的 `state` 和 `action` 了。



&emsp;&emsp;现在来看创建 `store` 这步，现在有了两个 `reducer` 来管理各自的 `state`，需要将它们合并成一个 `reducer` 来创建 `store`。

1. 在 `src/reducers/` 中新建一个 `index.js` 来封装合并后的 `reducer`：

```js
import { combineReducers } from 'redux';
import user from './user';
import counter from './counter';

const rootReducer = combineReducers({
    user, 
    counter,
});

export default rootReducer;
```

&emsp;&emsp;代码中的 `combineReducers` 方法中，`user` 其实是`user:user`，所以说，这个`reducer` 维护的 `state` 其实是一个对象。

2. 在 `src/index.js` 中使用这个合并的 `reducer` 创建 `store`：

```jsx
import rootReducer from './reducers';
// 创建 store 仓库
const store = createStore(rootReducer); 
```

3. 这样的话，需要返回 `src/App.js` 中修改一下之前的`state` 的映射 `mapStateToProps` :

```js
const mapStateToProps = (state) => {
    return {
        counter: state.counter,
        user: state.user,
    };
};
```

这样就完成了。



==有一个需要非常注意的地方==

&emsp;&emsp;在测试时发现，`reducer` 中，**对于 `state` 的更新操作需要返回一个新的 `state` 才会触发 `dispatch` 中监听的`state` 变化然后进行元素渲染。**

> 这个和三大原则的 `State 是只读的` 原则有关，若想改变 `state` 的话只能返回一个新的 `state`。

如：

```js
import * as actions from '../constants';

const user = ( state = [], action ) => {
    switch(action.type) {
        case actions.ADD_USER:
            // state.push(action.user);
            return state.concat(action.user);
        default:
            return state;
    }
};

export default user;
```

&emsp;&emsp;`array` 的 `concat` 方法是浅复制，并会返回一个**新**数组，因此能够正常的渲染：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723125100275.png" alt="image-20200723125100275" style="zoom:80%;" />

&emsp;&emsp;否则，若代码是这样的：

```js
case actions.ADD_USER:
    console.log('before state: ', state);
    state.push(action.user);
    console.log('after state: ', state);
    // return state.concat(action.user);
    return state;
```

&emsp;&emsp;点击 `addUser` 按钮，虽然在输出中发现`state` 是变化的，但是这些变化的 `state` 并不会及时渲染到页面上

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723125358933.png" alt="image-20200723125358933" style="zoom:80%;" />

&emsp;&emsp;然后要点击一次 `increment` 或 `decrement` 按钮才会触发渲染：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723125639079.png" alt="image-20200723125639079" style="zoom:80%;" />

&emsp;&emsp;因此要特别注意需要返回一个 **“新”** 的 `state`：

```js
case actions.ADD_USER:
    let newState = state.concat(action.user);
    return newState;
```

# 中间件

&emsp;&emsp;官方文档：https://www.redux.org.cn/docs/advanced/Middleware.html

<img src="https://gitee.com/ylea/imagehost/raw/master/img/OIP%20(1).jpg" alt="OIP (1)" style="zoom: 150%;" />

&emsp;&emsp;从图中可以看到 `redux` 的工作流，在触发 `actions` 后，到达 `store` 时，最开始会先经过一个 `minddleware`(中间件)，然后才通过 `reducer` 来更新状态。

> **它提供的是位于 action 被发起之后，到达 reducer 之前的扩展点。** 你可以利用 Redux middleware 来进行日志记录、创建崩溃报告、调用异步接口或者路由等等。

&emsp;&emsp;中间件通常可以用来打印日志、错误等。`redux` 的中间件可以自定义也可以使用别人开源的。

## 自定义中间件

&emsp;&emsp;代码中需要注意的是，中间件需要在创建 `store` 前编写，并在 `createStore` 中使用，如代码所示，中间件可以有多个，在`applyMiddleware` 中用逗号隔开即可。

&emsp;&emsp;`createStore` 中的第二个参数不知道是什么，先不用管它。

```jsx
import { createStore, applyMiddleware } from 'redux';

const logger = store => next => action => {
    console.log('dispatch -> ', action);
    let result = next(action); // 加载下一个中间件
    console.log('next state -> ', store.getState());
    return result;
};

const error = store => next => action => {
    try {
        next(action);
    } catch(e) {
        console.log('error -> ', e);
    }
};

// 创建 store 仓库
const store = createStore(rootReducer, {}, applyMiddleware(logger, error)); 
```

&emsp;&emsp;上面代码中的中间件定义使用了多重嵌套箭头函数的写法，其实本质是函数的嵌套，想象着用括号分隔可能会好理解一些：

```js
const logger = store => next => action => {};

function logger(store) {
    return function(next) {
        return function(action) {
            
        }
    }
}

const logger = (store => (next => (action => {})));
```



## 使用第三方中间件

&emsp;&emsp;日志的第三方中间件有：`redux-logger`

`cnpm i --save redux-logger`

```js
import logger from 'redux-logger';

const store = createStore(rootReducer, {}, applyMiddleware(logger)); 
```

&emsp;&emsp;效果比自定义的好看多了：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723165643383.png" alt="image-20200723165643383" style="zoom:80%;" />

## 异步中间件

`cnpm i --save redux-thunk`

目标：在点击 `increment` 按钮时，延时一秒获取数据：

&emsp;&emsp;在 `src/actions/counter.js` 中修改 `increment` 代码如下：

```js
export function increment(num) {
    return dispatch => {
        setTimeout(() => {
            dispatch({
                type: actions.INCREMENT,
                num
            });
        }, 1000);
    };
}
// src/index.js
import thunk from 'redux-thunk';
const store = createStore(rootReducer, {}, applyMiddleware(logger, thunk));
```

&emsp;&emsp;若未传入 `thunk` 中间件，会报这样的错：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723182830470.png" alt="image-20200723182830470" style="zoom:80%;" />

&emsp;&emsp;传入中间件 `thunk`并点击 `increment` 后：过一秒页面上的数字会变成 10

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723182007956.png" alt="image-20200723182007956" style="zoom:80%;" />

&emsp;&emsp;对于 `decrement`，也修改代码，不过代码中没有异步操作：

```js
export function decrement(num) {
    return dispatch => {
        dispatch({
            type: actions.INCREMENT,
            num
        });
    };
}
```

&emsp;&emsp;点击按钮后，也会显示：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723183655255.png" alt="image-20200723183655255" style="zoom:80%;" />

&emsp;&emsp;不过，若没有传入 `redux-thunk`，上面的两个代码都会报错。



**感觉下面的猜想有点问题，不过当看着玩帮助理解：**

&emsp;&emsp;结合下面的源码，**自己的猜想**：在触发 `action` 后，会先进入中间件，若有传入处理`action` 的中间件，则会先处理，最后再调用 `dispatch` 方法，再返回这个 `action` 的对象。

&emsp;&emsp;若未定义中间件，可能也会使用 `dispatch` 方法检测这个 `action` 是不是一个 `plain object`，若是，就返回这个 `action` 的对象，否则会报错。



&emsp;&emsp;`redux` 中关于 `dispatch` 的源码部分：

```js
/**
   * Dispatches an action. It is the only way to trigger a state change.
   *
   * The `reducer` function, used to create the store, will be called with the
   * current state tree and the given `action`. Its return value will
   * be considered the **next** state of the tree, and the change listeners
   * will be notified.
   *
   * The base implementation only supports plain object actions. If you want to
   * dispatch a Promise, an Observable, a thunk, or something else, you need to
   * wrap your store creating function into the corresponding middleware. For
   * example, see the documentation for the `redux-thunk` package. Even the
   * middleware will eventually dispatch plain object actions using this method.
   *
   * @param {Object} action A plain object representing “what changed”. It is
   * a good idea to keep actions serializable so you can record and replay user
   * sessions, or use the time travelling `redux-devtools`. An action must have
   * a `type` property which may not be `undefined`. It is a good idea to use
   * string constants for action types.
   *
   * @returns {Object} For convenience, the same action object you dispatched.
   *
   * Note that, if you use a custom middleware, it may wrap `dispatch()` to
   * return something else (for example, a Promise you can await).
   */


function dispatch(action) {
    if (!isPlainObject(action)) {
        throw new Error('Actions must be plain objects. ' + 'Use custom middleware for async actions.');
    }

    if (typeof action.type === 'undefined') {
        throw new Error('Actions may not have an undefined "type" property. ' + 'Have you misspelled a constant?');
    }

    if (isDispatching) {
        throw new Error('Reducers may not dispatch actions.');
    }

    try {
        isDispatching = true;
        currentState = currentReducer(currentState, action);
    } finally {
        isDispatching = false;
    }

    var listeners = currentListeners = nextListeners;

    for (var i = 0; i < listeners.length; i++) {
        var listener = listeners[i];
        listener();
    }

    return action;
}
```

# 使用 redux-thunk 的示例

**目标**：在原有基础上使用异步操作获取测试 API `http://iwenwiki.com/api/blueberrypai/getChengpinDetails.php` 中的某些内容：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200723204950304.png" alt="image-20200723204950304" style="zoom:80%;" />



&emsp;&emsp;根据下面的图，要实现这个功能，可以在触发 `action` 后，进入 `reducer` 更新状态之前获取数据，然后再`dispatch` 到 `reducer` 中进行更新，最后可以从 `store` 中获取 `state`。

<img src="https://gitee.com/ylea/imagehost/raw/master/img/OIP%20(1).jpg" alt="OIP (1)" style="zoom: 150%;" />

**1.** 首先，为 `user` 再定义一个常量来表示 `action.type`：

```js
// user
export const ADD_USER = 'ADD_USER';
export const FETCH_USER_DATA = 'FETCH_USER_DATA'; // 新添加的
```

**2.** 在`src/actions/user.js` 中新增 `user` 的 `action`：

```js
import * as actions from '../constants';

export function addUser(user) {
    return {
        type: actions.ADD_USER,
        user
    };
}

function fetch_user_data(userData) {
    return {
        type: actions.FETCH_USER_DATA,
        userData
    }
};

export function fetch_user() {
    return dispatch => {
        fetch('http://iwenwiki.com/api/blueberrypai/getChengpinDetails.php')
        .then(res => res.json())
        .then(data => {
            dispatch(fetch_user_data(data.chengpinDetails[0]));
        })
        .catch(error => {
            console.log('error: ', error);
        });
    };
}
```

&emsp;&emsp;需要注意的是，最后导出的是 `fetch_user`，在获取的数据后，需要将 `fetch_user_data` 方法中返回的 `action` 传递给 `dispatch`，这样才能正常的使用 `reducer` 来更新状态。



**3.** 在 `src/reducers/user` 中重新定义 `state`：

&emsp;&emsp;代码中定义了一个 `initUserInfo` 对象来作为 `user` 的 `state`。

&emsp;&emsp;注意到，两个`case`的返回结果都是一个新对象，而这个对象只会更新对应的数据，其他的使用原本的`state` 里的数据。 

```js
import * as actions from '../constants';

const initUserInfo = {
    user: [],
    userData: {}
};

const user = ( state = initUserInfo, action ) => {
    switch(action.type) {
        case actions.ADD_USER:
            let user = state.user.concat(action.user);
            return {
                user: user,
                userData: state.userData
            };
        case actions.FETCH_USER_DATA:
            return {
                user: state.user,
                userData: action.userData
            };
        default:
            return state;
    }
};

export default user;
```

**4.** 在 `App.js` 中进行修改：

&emsp;&emsp;在 `mapStateToProps` 中新增 `userData` 的映射，且注意到，现在的 `state` 里的 `user` 变成了一个对象了，可以按照下面的代码来与 `props` 进行映射，也可以直接用这个 `user` 对象来进行映射。

```js
const mapStateToProps = (state) => {
    return {
        counter: state.counter,
        user: state.user.user,
        userData: state.user.userData,
    };
};

```

&emsp;&emsp;然后，在 `User` 组件中多传入两个参数，一个参数是新增的获取到的数据，另一个是触发获取数据的方法：

```jsx
<User 
    users={this.props.user}
    userData={this.props.userData}
    addUser={() => { userActions.addUser('ye'); }}
    fetchUserData={() => (userActions.fetch_user())} 
    />
```

**5.** 最后在 `/src/component/User.jsx` 组件中使用这两个新增的属性：

```jsx
<div className='text-center'>
    <br />
    <h1>{this.props.users}</h1>
    <button onClick={this.props.addUser}>addUser</button>
    <br />
    <h3>{this.props.userData.title}</h3>
    <button onClick={this.props.fetchUserData}>fetcuUserData</button>
</div>
```

&emsp;&emsp;其实，不使用 `User` 组件传递 `props` 而是直接在 `User` 组件中进行与 `state` 和 `dispatch` 的绑定会更好些，因为 `redux` 的本意就是状态只由一个 `sotre` 来管理，因此修改一下 `User` 组件，当然在使用 `User` 组件的地方也需要修改，不过这边就不贴修改的代码了：

```jsx
import React, { Component } from 'react';
import { connect } from 'react-redux';
import * as userActions from '../actions/user';
import { bindActionCreators } from 'redux';

class User extends Component {
    
    render() {
        console.log(this.props);
        const { userActions } = this.props;
        return (
            <div className='text-center'>
                <br />
                <h1>{this.props.user}</h1>
                <button onClick={() => userActions.addUser('ye')}>addUser</button>
                <br />
                <h3>{this.props.userData.title}</h3>
                <button onClick={() => userActions.fetch_user()}>fetcuUserData</button>
            </div>
        );
    }
}

const mapStateToProps = (state) => {
    return {
        user: state.user.user,
        userData: state.user.userData
    }
};

const mapDispatchToProps = (dispatch) => {
    return {
        userActions: bindActionCreators(userActions, dispatch)
    };
};

export default connect(mapStateToProps, mapDispatchToProps)(User);
```

## Redux 示例优化

&emsp;&emsp;在上个示例中，有使用异步操作来获取数据，由于有网络请求，因此若网速不好的话获取数据可能会有延时，因此在未获取到数据时应该要显示加载提示，等获取到数据后再显示数据：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/2020-07-24-21-10-35.gif" alt="2020-07-24-21-10-35" style="zoom:80%;" />

**1.** 在常量中添加几个 `action.type` ：

```js
export const FETCH_USER_DATA_SUCCESS = 'FETCH_USER_DATA_SUCCESS';
export const FETCH_USER_DATA_REQUEST = 'FETCH_USER_DATA_REQUEST';
export const FETCH_USER_DATA_FALIURE = 'FETCH_USER_DATA_FALIURE';
```

**2.** 在 `src/actions/user.js` 中定义几个 `action` ，在`fetch` 数据之前，触发 `request`动作，当获取到数据后，触发 `success`动作，若遇到错误，则触发`faliure` 动作。

&emsp;&emsp;需要注意的是，在返回值是非纯对象的情况下，要想触发 `action` ，需要使用 `dispatch()` 方法：

```js
function fetch_user_data_success(userData) {
    return {
        type: actions.FETCH_USER_DATA_SUCCESS,
        userData
    }
};

function fetch_user_data_request() {
    return {
        type: actions.FETCH_USER_DATA_REQUEST,
    }
};

function fetch_user_data_faliure(error) {
    return {
        type: actions.FETCH_USER_DATA_FALIURE,
        error
    }
};

export function fetch_user() {
    return dispatch => {
        dispatch(fetch_user_data_request());
        fetch('http://iwenwiki.com/api/blueberrypai/getChengpinDetails.php')
        .then(res => res.json())
        .then(data => {
            dispatch(fetch_user_data_success(data.chengpinDetails[0]));
        })
        .catch(error => {
            dispatch(fetch_user_data_faliure(error));
            console.log('error: ', error);
        });
    };
}
```

**3.** 在 `src/reducers/user.js` 中新增几个 `action` 的对应操作：

```js
const initUserInfo = {
    user: [],
    userData: {},
    isFetching: false,
    error: null
};

const user = ( state = initUserInfo, action ) => {
    switch(action.type) {
        case actions.ADD_USER:
            let user = state.user.concat(action.user);
            return {
                user: user,
                userData: state.userData,
                isFetching: false,
                error: null
            };
        case actions.FETCH_USER_DATA_SUCCESS:
            return {
                user: state.user,
                userData: action.userData,
                isFetching: false,
                error: null
            };
        case actions.FETCH_USER_DATA_REQUEST:
            return {
                user: state.user,
                userData: {},
                isFetching: true,
                error: null
            };
        case actions.FETCH_USER_DATA_FALIURE:
            return {
                user: state.user,
                userData: {},
                isFetching: false,
                error: action.error
            };
        default:
            return state;
    }
};
```

**4.** 最后，在 `User.jsx` 中将 `reducer` 中定义的 `state` 里的数据映射到 `props` 上，然后再根据 `error` 和 `isFetching` 的值来选择屏幕上需要展示的数据：

```jsx
const mapStateToProps = (state) => {
    return {
        user: state.user.user,
        userData: state.user.userData,
        isFetching: state.user.isFetching,
        error: state.user.error
    }
};

render() {
        console.log(this.props);
        const { userActions } = this.props;
        const { user, userData, isFetching, error } = this.props;
        let data = '';
        if (error) {
            data = error;
        } else if (isFetching) {
            data = 'Loading...';
        } else {
            data = userData.title;
        }

        return (
            <div className='text-center'>
                <br /><br />
                <h1>{ user }</h1>
                <button  onClick={() => userActions.addUser('ye')}>addUser</button>
                <br /><br /><br />
                <h3>{ data }</h3><br />
                <button onClick={() => userActions.fetch_user()}>fetcuUserData</button>
            </div>
        );
    }
```



&emsp;&emsp;还有一个就是，若想要有网速慢的效果，可以在 Chrome 开发者工具那选择 3G 网速：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200724212108502.png" alt="image-20200724212108502" style="zoom:80%;" />

# Redux 调试工具

&emsp;&emsp;安装 `Redux` 的调试工具需要几个步骤：

1.  chrome 中需要安装插件： Redux DevTools

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200724212817540.png" alt="image-20200724212817540" style="zoom:80%;" /> 



刚开始是这样的：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200724213017200.png" alt="image-20200724213017200" style="zoom:80%;" />

也能在拓展栏中显示：

<img src="Redux.assets/image-20200724214013320.png" alt="image-20200724214013320" style="zoom:80%;" />



2. 安装插件的对应依赖：`cnpm i --save redux-devtools-extension`

3. 在 主文件入口中使用（即创建 `store` 的文件中）：

   ```js
   // 引入依赖方法
   import { composeWithDevTools } from 'redux-devtools-extension';
   
   // 包裹在原先中间件的位置：
   const store = createStore(rootReducer, {}, composeWithDevTools(applyMiddleware(logger, thunk))); 
   ```

&emsp;&emsp;然后就可以在页面中查看效果了：原先插件的图标会变亮，右键中也会有插件的图标

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200724213644211.png" alt="image-20200724213644211" style="zoom:80%;" />

&emsp;&emsp;最底部的按钮能够一步步还原 `action` 的触发步骤，且效果也能在页面上实时显示：

<img src="https://gitee.com/ylea/imagehost/raw/master/img/image-20200724213817938.png" alt="image-20200724213817938" style="zoom: 67%;" />





