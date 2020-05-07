React 学习记录
---

> 跟随 React 官网 Tic Tac Toe 教程学习

#### 什么是React？

React 是一个声明式的、高效的，并且灵活的用于构建用户界面的 JavaScript 库。它允许您使用 ”components“（小巧而独立的代码片段）组合出各种复杂的 UI。



#### React 组件类型

从认识 React.Component 开始。

示例代码：

```js
class ShoppingList extends React.Component {
  render() {
    return (
      <div className="shopping-list">
        <h1>Shopping List for {this.props.name}</h1>
        <ul>
          <li>Instagram</li>
          <li>WhatsApp</li>
          <li>Oculus</li>
        </ul>
      </div>
    );
  }
}
```

ShoppingList 是一个 **React 组件类** ，或 **React 组件类型**。组件接收参数，称为 `props`(属性)，并通过 `render` 方法返回一个显示的视图层次结构。

`render` 方法返回要渲染的内容*描述*，然后 React 接受该描述并将其渲染到屏幕上。特别是，`render` 返回一个 **React 元素**，这是一个渲染内容的轻量级描述。大多数 React 开发人员使用一种名为 JSX 的特殊语法，可以更容易地编写这些结构。`<div />` 语法在构建时被转换为 `React.createElement('div')`。上面的例子等价于：

```javascript
React.createElement("div", {
  className: "shopping-list"
}, React.createElement("h1", null, "Shopping List for ", props.name), React.createElement("ul", null, React.createElement("li", null, "Instagram"), React.createElement("li", null, "WhatsApp"), React.createElement("li", null, "Oculus")));
```

可以将任何 JavaScript 表达式放在 JSX 中的大括号内。每个 React 元素都是一个真正的 JavaScript 对象，可以将其存储在变量中或传递给程序。

`ShoppingList` 组件仅渲染内置的 DOM 组件，但可以通过使用 `<ShoppingList />` 轻松地编写自定义的 React 组件。每个组件都是封装的，因此它可以独立操作，从简单的组件构建复杂的UI。



### 通过 Props 传递数据



































