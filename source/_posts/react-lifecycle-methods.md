---
title: 「译」如何使用React生命周期方法
date: 2018-07-28 13:39:28
tags: React
comments: true
categories: "React"
---

> 原文链接：[How to use React Lifecycle Methods](https://www.andreasreiterer.at/reactjs-lifecycle-methods/)

## React 生命周期

大体上分三类

- Mount
- Update
- Unmount

<!--more-->

### Mount

- _constructor()_
- _~~componentWillMount()~~_
- _render()_
- _componentDidMount()_

### Update

- _~~componentWillReceiveProps()~~ / static getDerivedStateFromProps()_
- _shouldComponentUpdate()_
- _~~componentWillUpdate()~~ / getSnapshotBeforeUpdate()_
- _render()_
- _componentDidUpdate()_

### Unmount

- _componentWillUnmount()_

## 生命周期方法

### componentWillMount（即将移除）

```javascript
componentWillMount()
```

当 React 渲染一个组件的是你，首先进入该方法。

**Note：**`componentWillMount()`是唯一一个在`render()`之前调用的生命周期方法。因此是在服务端渲染中唯一被调用的方法。

因为`componentWillMount()`将被删除，所以官方推荐使用`constructor()`替代该方法

**Update：**该方法应该在新的代码中避免使用

#### State

可以在该方法中使用`this.setState()`但是不一定触发重新渲染。

### componentDidMount

```javascript
componentDidMount()
```

当该方法被调用时候，React 已经渲染了组件并且将组件插入 DOM。因此如有有任何依赖于 DOM 的初始化，应该放在这里。

#### State

该方法中可以使用`this.setState()`方法，它将触发重新渲染。

#### Use Cases

- fetch data
- 依赖 DOM 初始化
- 添加事件监听

### componentWillReceiveProps（即将移除）

```javascript
componentWillReceiveProps(nextProps)
```

当组件接收到新的`props`，该方法会首先被调用，但是需要注意一点，**即使`props`并没有发生改变，该方法也会被调用**，所以使用该方法的时候要确保比较`this.props`和`nextProps`，避免不必要的渲染。

**Update：**`componentWillReceiveProps`将被移除，使用新的`static getDerivedStateFromProps`代替。

### static getDerivedStateFromProps（新增）

```javascript
static getDerivedStateFromProps(props, state)
```

每次`render`都会调用该方法——即使`props`没有发生变化。**所以要谨慎使用。**

### shouldComponentUpdate

```javascript
shouldComponentUpdate(nextState, nextProps)
```

有些时候需要避免不必要的渲染，可以使用该方法。返回`false`意味着 React 不执行`componentWillUpdate()`，`render()`，`componentDidUpdate()`。

该方法在初始化时候不执行。

**Note：**根据 React 文档，React 可能将`shouldComponentUpdate`视做提示而不是严格的根据它的返回结果决定是否执行，也就是说可能出现`shouldComponentUpdate`返回`false`，但是还是发生重新渲染。

#### State

在该方法中不可以设置`setState`。

#### Use Case

如前所述，该方法可能有有问题。React 官方提供了另一个解决办法，所以如果发现某个组件慢，可以使用`React.PureComponent`替代`React.component`，它将对`props`和`state`提供一个浅层对照。

### componentWillUpdate（即将移除）

```javascript
componentWillUpdate(nextProps, nextState)
```

该方法在被渲染前调用。`shouldComponentUpdate`在新的`props`进入组件或者`state`改变的时候调用。

该方法在初始渲染时候不被调用。

**Update：**`shouldComponentUpdate`即将被移除。

#### State

在该方法中不能调用`setState`。

#### Use Cases

`shouldComponentUpdate`方法在`render()`前会被准确调用，所以在该方法中做任何跟 DOM 相关的操作是不合适的，因为很快会过期。

常见的用例有：

- 根据`state`的变化设置变量
- 派发事件
- 开始动画

### getSnapshotBeforeUpdate

```javascript
getSnapshotBeforeUpdate(prevProps, prevState)
```

该方法在 React 16.3 被添加并且它配合`componentDidUpdate`。该方法应该覆盖了旧方法`shouldComponentUpdate`的所有用例。

`getSnapshotBeforeUpdate`在元素被渲染并写入 DOM 之前调用，这样，你在 DOM 更新前捕获 DOM 信息（例如：滚动位置）。

应该返回一个`snapshot`值或`null`，无论返回什么，`shouldComponentUpdate`都可以接收到`snapshot`参数。

#### Use Cases

如果想要获得一个 list 或者一个聊天窗口中的滚动位置，可以在`getSnapshotBeforeUpdate`中取到这些信息。然后将滚动信息作为`snapshot`值返回。这允许在`shouldComponentUpdate`中是设置正确的滚动位置在 DOM 更新后。

### componentDidUpdate

```javascript
componentDidUpdate(prevProps, prevState, snapshot)
```

React 更新组件后，调用`componentDidUpdate`。该方法在初始渲染时候不会被调用。

#### Use Cases

如果组件更新后需要操作 DOM，可以使用该方法。

### componentWillUnmount

```javascript
componentWillUnmount()
```

在卸载，销毁组件之前调用该方法。

#### State

在卸载组件时候不能设置 State

#### Use Cases

使用该方法清理 actions。

- 删除在`componentDidMount`或其他地方添加的事件监听
- 断开网络请求
- 清空计时器
- 清理在`componentDidMount`中创建的 DOM 元素
