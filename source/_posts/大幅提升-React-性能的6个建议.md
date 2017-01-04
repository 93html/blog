---
title: 大幅提升 React 性能的6个建议
date: 2016-12-24 14:19:21
categories: 技术
tags: React
---
React 在不做任何优化的情况下性能也非常好，然而通过一些小小的优化，可以让性能进一步提升，通过以下这6条建议，可以数十倍加快渲染速度

## 设置 NODE_ENV 为 Production
React 在开发环境下，有完整的警告和错误检查，但它们不是为生产环境准备的，如果你看过 React 的源码，你会看到很多 `if (process.env.NODE_ENV != 'production')`，这些代码对于最终用户是不需要的，而且访问 `process.env.NODE_ENV` 会非常慢，对于生产环境而言，完全可以移除这些代码

如果你使用 [Webpack](https://webpack.github.io/)，你可以使用 [DefinePlugin](https://webpack.github.io/docs/list-of-plugins.html#defineplugin) 来替换 `process.env.NODE_ENV` 为 'production'，然后使用 [UglifyJsPlugin](https://webpack.github.io/docs/list-of-plugins.html#uglifyjsplugin) 移除这些不会执行的代码
``` js
// webpack.config.js
...
plugins: [
    new webpack.DefinePlugin({
        'process.env.NODE_ENV': JSON.stringify('production')
    }),
    new webpack.optimize.UglifyJsPlugin({
        compress: {
            warnings: false
        }
    })
]
...
```

## React 15 的渲染速度比 0.14 快约 25%
在 [React 15 的更新](https://facebook.github.io/react/blog/2016/04/07/react-v15.html)中非常重要的一项是，使用在现代化浏览器中性能更好的 `document.createElement` 替换 `innerHTML`，这一改动也意味着 React 将不再支持 IE8

## Babel Constant 和 Inline Elements 转换
Babel 为开发者们提供了 [React Constant Elements](http://babeljs.io/docs/plugins/transform-react-constant-elements/) 和 [React Inline Elements](https://babeljs.io/docs/plugins/transform-react-inline-elements/)，这两款插件能够在编译阶段将代码转换成更高效的形式，注意仅将它们用于生产环境

## 封装集合渲染为独立组件
这一点在循环渲染集合组件时尤其重要，React 在渲染大型集合是性能十分糟糕，原因是 React 会在每次更新中全部重新渲染，因此建议将渲染集合的部分装为独立的组件渲染
```js
// Bad
class MyComponent extends Component {
    render() {
        const {todos, user} = this.props;
        return (<div>
            {user.name}
            <ul>
                {todos.map(todo => <TodoView todo={todo} key={todo.id} />)}
            </ul>
        </div>)
    }
}
```
``` js
// Good
// 当 user.name 更新时，列表不会重新渲染
class MyComponent extends Component {
    render() {
        const {todos, user} = this.props;
        return (<div>
            {user.name}
            <TodosView todos={todos} />
        </div>)
    }
}

class TodosView extends Component {
    render() {
        const {todos} = this.props;
        return (<ul>
            {todos.map(todo => <TodoView todo={todo} key={todo.id} />)}
        </ul>)
    }
}
```

## 尽早绑定方法
在 render() 中绑定的方法应该尽早声明，而不是在渲染时定义
``` js
// Bad
render() {
    return <MyWidget onClick={() => { alert(this.state.text) }} />
}
```
``` js
// Good
constructor() {
    this.handleClick = this.handleClick.bind(this);
}

handleClick() {
    alert(this.state.text);
}

render() {
    return <MyWidget onClick={this.handleClick} />
}
```

## 不变组件禁用更新
对于不需要更新的组件，可以在 `shouldComponentUpdate()` 中 `return false`，或者使用 [Stateless Component](https://facebook.github.io/react/docs/components-and-props.html)
``` js
// Bad
class Logo extends Component {
    render() {
        return <div><img src='logo.png' /></div>;
    }
}
```

``` js
// Good
class Logo extends Component {
    shouldComponentUpdate() {
        return false;
    }

    render() {
        return <div><img src='logo.png' /></div>;
    }
}

// or Stateless Component
const Logo = () => <div><img src='logo.png' /></div>;
```

##### 参考文章
- [React performance](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md)
- [How to Make Your React Apps 15x Faster](https://reactjsnews.com/how-to-make-your-react-apps-10x-faster)
- [Avoid bind when passing props](https://daveceddia.com/avoid-bind-when-passing-props/)
- [Optimizing rendering React components](https://mobxjs.github.io/mobx/best/react-performance.html)
