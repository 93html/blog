---
title: 使用 HOC 抽象 React 组件逻辑
date: 2017-07-01 12:12:36
categories: 技术
tags: react
---

阅读本文前建议先对 react 高阶组件（HOC），以及 js 装饰器（Decorator）有基本的了解

首先来看这样2个组件（注意 clickHandler 部分）：

* CheckBox
```js
class CheckBox extends Component {
  state = {
    checked: false
  };
  
  render() {
    const { checked } = this.state;

    return (
      <div
        classNames={classnames({ active: checked })}
        onClick={this.clickHandler} />
    );
  }

  clickHandler = () => {
    this.setState({
      checked: !this.state.checked
    });
  }
}
```

* Dropdown List
```js
class DropDownList extends Component {
  state = {
    open: false
  };
  
  render() {
    const { open } = this.state;

    return (
      <div>
        <button onClick={this.clickHandler}>性别</button>
        {
          open &&
          <ul>
            <li>男</li>
            <li>女</li>
          </ul>
        }
      </div>
    );
  }

  clickHandler = () => {
    this.setState({
      open: !this.state.open
    });
  }
}
```
看出问题了吗？这两个组件有着几乎一样的 clickHandler 方法！

有没有方法把这段逻辑抽象出来呢？这里介绍一种使用高阶组件的抽象方式

### 使用 HOC 重构

通过观察，以上这2个组件，基本上就是一个状态控制开和关的逻辑，那么我们可以做一个 Decorator
```js
const toggleHOC = Wrapper => class extends Component {
  state = { on: false };

  render() {
    const { on } = this.state;
    
    return <Wrapper on={on} onToggle={this.toggleHandler} />;
  }

  toggleHandler = () => {
    this.setState({ on: !this.state.on });
  }
};
```
参数 Wrapper 是一个 React 组件，我们为这个组件的 props 注入2个属性，

* 一个布尔值的 on
* 一个触发修改 on 的方法 toggleHandler

最终返回出一个新的组件

再来修改 CheckBox 和 DropDownList：

```js
@toggleHOC
class CheckBox extends Component {
  render() {
    const { on, toggleHandler } = this.props;

    return (
      <div
        classNames={classnames({ active: on })}
        onClick={toggleHandler} />
    );
  }
}

@toggleHOC
class DropDownList extends Component {
  render() {
    const { on, toggleHandler } = this.props;

    return (
      <div>
        <button onClick={toggleHandler}>性别</button>
        {
          on &&
          <ul>
            <li>男</li>
            <li>女</li>
          </ul>
        }
      </div>
    );
  }
}
```
完成了！是不是很简单

HOC 还有很多高级用法，比如有这样的

```js
@foo({ a: 1, b: 2, c: 3 }) 
class A extends Component {...}
```
和这样的
```js
const abcHOC = compose(a, b, c);

@abcHOC
class A extends Component {...} 
```
这些用法以后再讲
