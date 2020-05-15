React 学习记录
---

> 跟随 React 官网 Tic Tac Toe 教程学习

### 一、什么是React？

React 是一个声明式的、高效的，并且灵活的用于构建用户界面的 JavaScript 库。它允许您使用 ”components“（小巧而独立的代码片段）组合出各种复杂的 UI。



### 二、React 组件类型

#### （1）组件声明

从认识 React.Component 开始。从定义上来说， 组件就像 JavaScript 的函数。组件可以接收任意输入(称为”props”)， 并返回 React 元素，用以描述屏幕显示内容。

最简单的有效的 React 组件：

```javascript
function Welcome(props) {
    return <h1>Hello, {props.name}</h1>;
}
```

可以用一个 [ES6 的 class](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Classes) 来定义一个组件：

```js
class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}

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

ShoppingList 是一个 **React 组件类**。组件接收参数，称为 `props`(属性)，并通过 `render` 方法返回一个显示的视图层次结构。

`render` 方法返回要渲染的内容*描述*，然后 React 接受该描述并将其渲染到屏幕上。特别是，`render` 返回一个 **React 元素**，这是一个渲染内容的轻量级描述。大多数 React 开发人员使用一种名为 JSX 的特殊语法，可以更容易地编写这些结构。`<div />` 语法在构建时被转换为 `React.createElement('div')`。上面的例子等价于：

```javascript
React.createElement("div", {
  className: "shopping-list"
}, React.createElement("h1", null, "Shopping List for ", props.name), React.createElement("ul", null, React.createElement("li", null, "Instagram"), React.createElement("li", null, "WhatsApp"), React.createElement("li", null, "Oculus")));
```

可以将任何 JavaScript 表达式放在 JSX 中的大括号内。每个 React 元素都是一个真正的 JavaScript 对象，可以将其存储在变量中或传递给程序。

`ShoppingList` 组件仅渲染内置的 DOM 组件，但可以通过使用 `<ShoppingList />` 轻松地编写自定义的 React 组件。每个组件都是封装的，因此它可以独立操作，从简单的组件构建复杂的UI。

#### （2）组件渲染

当 React 遇到一个代表用户定义组件的元素时，它将 JSX 属性以一个单独对象的形式传递给相应的组件。 我们将其称为 “props” 对象。

比如, 以下代码在页面上渲染 “Hello, Luwx” ：

```javascript
class Welcome extends React.Component {
    render() {
        return <h1>Hello, {this.props.name}</h1>;
    }
}

const element = <Welcome name="Luwx" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```

虽然 React 很灵活，但是它有一条严格的规则：

**所有 React 组件都必须是纯函数，并禁止修改其自身 props 。**



### 三、组件间通过 Props 传递数据

从 Board(棋盘) 组件传递一些数据到 Square(方格) 组件。在 Board(棋盘) 组件的 `renderSquare` 方法中，以 `value` prop(属性) 传递给 Square(方格) 组件：

```javascript
class Board extends React.Component {
    renderSquare(i) {
        // 渲染一个 Square 组件并传递参数
        return <Square value={i} />;
    }
    render() {
        const status = 'Next player: X';
        return (
            <div>
                <div className="status">{status}</div>
                <div className="board-row">
                    {this.renderSquare(0)}
                    {this.renderSquare(1)}
                    {this.renderSquare(2)}
                </div>
                <div className="board-row">
                    {this.renderSquare(3)}
                    {this.renderSquare(4)}
                    {this.renderSquare(5)}
                </div>
                <div className="board-row">
                    {this.renderSquare(6)}
                    {this.renderSquare(7)}
                    {this.renderSquare(8)}
                </div>
            </div>
        );
    }
}
```

然后修改 Square(方格) 的`render`方法，通过使用 `{this.props.value}` 来读取并显示该值：

```javascript
class Square extends React.Component {
    render() {
        return (
            <button className="square">
                {this.props.value}
            </button>
        );
    }
}
```



### 四、制作交互式组件

#### （1）组件响应动作

添加响应函数 onClick 响应点击动作：

```javascript
<button className="square" onClick={() => alert('click')}>
    {this.props.value}
</button>
```

请注意如何使用  `onClick={() => alert('click')}` ，我们将传递 *一个函数* 作为 `onClick` prop(属性)。它只在点击后触发。忘记 `() =>` 并直接编写 `onClick={alert('click')}` 是一个常见错误，并且每次组件重新渲染时都会触发，且只会触发一次，即不会响应方格点击动作。

#### （2）组件状态

React 组件可以通过在构造函数中设置 `this.state` 来拥有 **state(状态)** ，构造函数应该被认为是组件的**私有**。 让我们在 Square(方格) 组件的 state(状态) 中存储当前值，并在单击方格时更改它。

**调用 this.setState() 函数后会重新渲染组件，唯一可以直接设置 this.state 的地方是构造函数。**

首先，在类中添加一个构造函数来初始化 state(状态)：

```javascript
constructor() {
    super();
    this.state = {
        value: null,
    };
}
```

> 注意：
>
> 在 JavaScript classes 中， 在定义子类的构造函数时，需要始终调用 `super` 。所有具有  `constructor`  的 React 组件类都应该以 `super(props)` 调用启动它。

更改 Square(方格) 组件的 `render` 方法以显示当前 state(状态) 的值，并在点击时切换：

- 将 `<button>` 标签中的 `this.props.value` 替换为 `this.state.value` 。
- 用 `() => this.setState({value: 'X'})` 替换事件处理程序中的 `() => alert()` 。

在这些更改之后， Square(方格) 的`render`方法返回的 `<button>` 标记如下所示：

```javascript
<button
    className="square"
    onClick={() => this.setState({ value: 'X' })}
>
    {this.state.value}
</button>
```

### 五、状态提升

**要从多个子级收集数据 或 使两个子组件之间相互通信，需要在其父组件中声明共享 state(状态) 。父组件可以使用props(属性) 将 state(状态) 传递回子节点；这可以使子组件彼此同步并与父组件保持同步。当重构 React 组件时，提升 state(状态) 是非常常见的。**

#### （1）子组件状态交由父组件控制

虽然 Board(棋盘) 应该可以获取到每个 Square(方格) 的当前 state(状态) ，但是这么做往往使代码难以理解，更脆弱，更难重构。相反，最好的解决办法是将 state(状态) 存储在 Board(棋盘) 组件中，而不是在每个 Square(方格) 组件中 ，Board(棋盘) 组件可以通过传递 props(属性) 来告诉每个 Square(方格) 组件要显示什么。

在 Board(棋盘) 组件中添加一个构造函数，并设置其初始 state(状态) 为包含一个具有 9 个空值的数组，对应 9 个方格：

```javascript
constructor() {
    super();
    this.state = {
        squares: Array(9).fill(null),
    }
}
```

再次使用 prop(属性) 传递机制。 修改 Board(棋盘) ，以指示每个 Square(方格) 的当前值（“X”，“O’”或“null”）。 我们已经在 Board(棋盘) 的构造函数中定义了 `squares` 数组， 我们将修改 Board(棋盘) 的 `renderSquare` 方法来读取它：

```javascript
renderSquare(i) {
    return <Square value={this.state.squares[i]} />;
}
```

每个 Square(方格) 现在都会收到一个 `value` prop(属性)，它要么是 `'X'`，要么是`'O'`，对于空 Square(方格) 则是`null`。

接下来，我们需要修改当点击方格的时候会发生的事情。Board(棋盘) 组件现在存储了那些已经填充的方格，这意味着我们需要一些方法来使  Square(方格) 组件 更新 Board(棋盘) 组件的 state(状态) 。由于组件的 state(状态) 被认为是私有的，我们不能从  Square(方格) 组件直接更新 Board(棋盘) 的 state(状态) 。

通常的模式是将一个**函数**从 Board(棋盘) 组件 传递到 Square(方格) 组件上，该函数在方格**被点击**时调用。 再次修改 Board(棋盘) 组件中 `renderSquare()` ，修改为：

```javascript
// 把 Board(棋盘) 组件中 2 个的 props(属性) 传递给 Square(方格) 组件：value 和 onClick。onClick prop(属性) 是一个函数，Square(方格) 组件 可以调用该函数。
renderSquare(i) {
    // 渲染一个 Square 组件并传递参数
    return (
        <Square
            value={this.state.squares[i]}
            onClick={() => this.handleClick(i)}
        />
    );
}
```

> 注意：
>
> 将返回的元素拆分成多行以提高可读性，并在其两边加上括号，这样 JavaScript 就不会在 `return` 后插入分号并且破坏我们的代码了。

修改整个 Square(方格) 组件，获取 Board(棋盘) 组件传递的数据和方法：

```javascript
class Square extends React.Component {
    render() {
        return (
            <button
                className="square"
                onClick={() => this.props.onClick()}
            >
                {this.props.value}
            </button>
        );
    }
}
```

> 注意：
>
> DOM `<button>` 元素的 `onClick` 属性对于 React 来说有特殊意义，因为它是一个内置组件。 对于像 Square(方格) 这样的自定义组件，命名取决于您自己。我们可以用不同的方式命名  Square(方格) 的`onClick` prop(属性) 或  Board(棋盘)  的 `handleClick` 方法。在React中，对于表示事件的 props(属性) 使用 `on[Event]` 命名，对处理事件的方法使用 `handle[Event]` 。

#### （2）组件间函数调用链路

现在当方格被点击时，它调用由 Board(棋盘) 组件传递的 `onClick` 函数，调用链路如下：

1. 内置 DOM 的 `<button>` 组件上的 `onClick` prop(属性) 告诉 React 设置一个 click 事件侦听器。
2. 当点击按钮时，React 将调用在 Square(方格) 组件 `render()` 方法中定义的 `onClick` 事件处理程序。
3. 这个事件处理程序调用 `this.props.onClick()` 。 Square(方格) 组件的 `onClick`  props(属性) 由 Board(棋盘) 组件指定。
4. Board(棋盘) 组件将 `onClick={() => this.handleClick(i)}` 传递给 Square(方格) 组件，所以当被调用时，它会在 Board(棋盘) 组件 上运行 `this.handleClick(i)` 。
5. 我们还没有在 Board(棋盘) 组件 上定义 `handleClick()` 方法，所以导致代码崩溃。

> 想要在 `this.handleClick(i)` 中写上 `this.setState(i)` 来直接更新 Square(方格) 组件的状态，这种做法不成立，因为 `this` 指向的是 Board(棋盘) 组件。

在 Board(棋盘) 组件中添加 `handleClick` 函数：

```javascript
handleClick(i) {
    // 创建副本
    const squares = this.state.squares.slice();
    squares[i] = 'X';
    // 更新状态并刷新子组件
    this.setState({squares: squares});
}
```

在这些修改之后，我们就能够再次点击 Squares(方格) 来填充它们。 但是，现在 state(状态) 已经存储在  Board(棋盘)  组件中而不是单个 Square(方格) 组件中。**当 Board(棋盘) 的状态发生变化时， Square(方格) 组件会自动重新渲染。** 保持 Board(棋盘) 组件中所有 Square(方格) 组件的 state(状态) 将允许它在将来确定胜利者。

由于 Square(方格) 组件不再保存 state(状态) ， Square(方格) 组件从 Board(棋盘) 组件接收值，并在单击它们时通知 Board(棋盘) 组件。 在 React 术语中， Square(方格) 组件现在是**受控组件**。 Board(棋盘) 会完全控制他们。



### 六、不可变数据的重要性

通常有两种方式来更改数据。第一种方法是通过直接更改变量的值来 *改变* 数据。第二种方法是用包含所需更改对象的新副本来替换数据。

#### （1）不通过赋值改变数据

```javascript
var player = {score: 1, name: 'Jeff'};

var newPlayer = Object.assign({}, player, {score: 2});
// 现在 player 没改变, 但是 newPlayer 是 {score: 2, name: 'Jeff'}

// 或者如果使用对象扩展语法，可以写成：
// var newPlayer = {...player, score: 2};
```

通过不直接改变数据（或更改底层数据）有一个额外的好处，可以帮助我们增强组件和整体应用性能。

#### （2）检测变更

检测可变对象的变化很困难，因为它们是直接修改的。该检测需要将可变对象与其自身的先前副本进行比较， 以及要遍历的整个对象树。

检测不可变对象中的更改要容易得多。 如果被引用的不可变对象与前一个不同，则该对象已更改。仅此而已。

#### （3）确定何时重新渲染

React 中不可变数据最大好处在于当构建简单的 *纯(pure)组件* 时。由于不可变性(Immutability) 可以更容易地确定是否已经进行了更改，这也有助于确定组件何时需要重新渲染。



### 七、函数式组件

现在将 Square(方格) 改为 **函数式组件（Functional Components）** 。

在 React 中，**函数式组件（Functional Components）**是一种更简单的方法来编写**只包含 `render` 方法且没有自己 state(状态) 的组件**。 而不是定义一个 `React.Component` 的扩展类， 我们可以编写一个函数，它将 `props` 作为输入，并返回应该渲染的内容。 **函数式组件**写入比类更乏味， 并且可以用这种方式表达许多组件。

用这个函数替换  Square(方格)  类：

```javascript
function Square(props) {
    return (
        <button
            className="square"
            onClick={props.onClick}
        >
            {props.value}
        </button>
    );
}
```

> 注意：
>
> 当我们将 Square 修改为一个函数式组件时，我们还将 `onClick={() => this.props.onClick()}` 更改为更短的`onClick = {props.onClick}`（注意两边都去掉了括号）。 在类中，我们使用箭头函数来访问正确的 `this` 值，但在函数式组件中我们不需要担心 `this` 。



### 八、状态切换（轮流下棋）

在 Board(棋盘) 组件中添加标志位记录选手，并在 `handleClick` 函数中切换：

```javascript
constructor() {
    super();
    this.state = {
        squares: Array(9).fill(null),
        // 选手标记
        xIsNext: true,
    }
}

handleClick(i) {
    // 创建副本
    const squares = this.state.squares.slice();
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    // 更新状态并刷新子组件
    this.setState({
        squares: squares,
        // 反转状态，切换选手
        xIsNext: !this.state.xIsNext,
    });
}
```

> 记得修改 `render()` 中的提示。

最后在 Board(棋盘) 组件的 `render()` 和 `handleClick()` 函数中通过 `this.state.squares` 属性判断胜负即可。



### 九、游戏过程回放

如果我们改变了 `square` 数组，实现过程回放将非常困难。

但是，我们使用 `slice()` 在每次移动后创建 `squares` 数组的新副本， 并**将其视为不可变的**。 这将允许我们存储 `square` 数组的每个过去版本， 并在已经发生的玩家轮换之间切换。

我们将过去的 `square` 数组存储在另一个名为 `history` 的数组中。 `history` 数组代表所有 Board(棋盘) 的 state(状态)， 从第一步到最后一步， 并有这样的数据：

```javascript
history = [
  // Before first move
  {
    squares: [
      null, null, null,
      null, null, null,
      null, null, null,
    ]
  },
  // After first move
  {
    squares: [
      null, null, null,
      null, 'X', null,
      null, null, null,
    ]
  },
  // After second move
  {
    squares: [
      null, null, null,
      null, 'X', null,
      null, null, 'O',
    ]
  },
  // ...
]
```

#### （1）再次状态提升

我们希望 Game(游戏) 组件显示过去动作列表。 它需要访问 `history` 来做到这一点， 所以我们将 `history` state(状态) 放在顶级 Game(游戏) 组件中。

将 `history` state(状态) 放入 Game(游戏) 组件可以让我们从子 Board(棋盘) 组件中删除 `squares` state(状态) 。 就像我们将  Square(方格)组件 中的 “state(状态)提升” 到 Board(棋盘)组件一样，我们现在将其从 Board(棋盘) 组件 提升到顶级 Game(游戏) 组件中。 这使 Game(游戏) 组件可以完全控制 Board(棋盘) 的数据， 并让它根据 `history` 显示 Board(棋盘) 的纪录。

首先，通过添加一个构造函数来设置 Game(游戏) 组件的初始 state(状态) ：

```javascript
class Game extends React.Component {
    constructor() {
        super();
        this.state = {
            history: [{
                squares: Array(9).fill(null),
            }],
            xIsNext: true,
        };
    }
}
```

接下来将让 Board(棋盘) 组件从Game(游戏) 组件接收 `squares` 和 `onClick` prop(属性) 。 因为我们现在在 Board(棋盘) 中有一个单击处理程序用于许多 Square(方格) ， 我们需要将每个 Square(方格) 的索引位置传递给 `onClick` 处理程序，以指示单击了哪个 Square(方格) 。 以下是转换 Board(棋盘) 组件所需的步骤：

- 删除 Board(棋盘) 组件中的 `constructor` (构造函数)。
- 在 Board(棋盘) 组件的 `renderSquare` 中用 `this.props.squares[i]` 替换 `this.state.squares[i]` 。
- 在 Board(棋盘) 组件的 `renderSquare` 中用 `this.props.onClick(i)` 替换 `this.handleClick(i)` 。

现在 Game(游戏) 组件的 `render` 应该可以获取最近的历史记录，并可以接管计算游戏状态：

```javascript
class Game extends React.Component {
    constructor() {
        super();
        this.state = {
            history: [{
                squares: Array(9).fill(null),
            }],
            xIsNext: true,
        };
    }

    handleClick(i) {
        // 创建副本
        const history = this.state.history;
        const current = history[history.length - 1];
        const squares = current.squares.slice();

        // “胜负已分”或“方格已经有棋子”则停止响应点击事件
        if (calculateWinner(squares) || squares[i]) {
            return;
        }
        
        // 更新状态并刷新子组件
        squares[i] = this.state.xIsNext ? 'X' : 'O';
        this.setState({
            history: history.concat([{
                squares: squares,
            }]),
            // 反转状态，切换选手
            xIsNext: !this.state.xIsNext,
        });
    }
    
    render() {
        const history = this.state.history;
        const current = history[history.length - 1];
        const winner = calculateWinner(current.squares);

        let status;
        if (winner) {
            status = 'Winner: ' + winner;
        } else {
            status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
        }

        return (
            <div className="game">
                <div className="game-board">
                    <Board 
                        squares={current.squares}
                        onClick={(i) => this.handleClick(i)}
                    />
                </div>
                <div className="game-info">
                    <div>{status}</div>
                    <ol>{/* TODO */}</ol>
                </div>
            </div>
        );
    }
}
```

> Javascript Array 的 `push()` 方法会改变原来的数组，而 `concat()` 方法不会。

#### （2）显示过去的动作

JavaScript 数组有一个 `map() `方法，它通常用于映射数据到其他数据，例如：

```javascript
// array.map( function(currentValue, index, arr), thisValue )
const numbers = [1, 2, 3];
const doubled = numbers.map(x => x * 2); // [2, 4, 6]
```

使用 `map` 方法，我们可以将我们的下棋动作历史映射到屏幕中按钮表示的 React 元素上， 并显示一个按钮列表以“跳转”到过去的动作。

在 Game(游戏) 组件的 `render` 方法中对 `history` 进行 `map` ：

```javascript
render() {
    const history = this.state.history;
    const current = history[history.length - 1];
    const winner = calculateWinner(current.squares);

    // 数组映射跳转，step 为 history 内的 {squares}，move 为遍历的 index
    const moves = history.map((step, move) => {
        const desc = move ? 'Go to move #' + move : 'Go to game start';

        return (
            <li>
                <button onClick={() => this.jumpTo(move)}>{desc}</button>
            </li>
        );
    });
	
    let status;
    if (winner) {
        status = 'Winner: ' + winner;
    } else {
        status = 'Next player: ' + (this.state.xIsNext ? 'X' : 'O');
    }

    return (
        <div className="game">
            <div className="game-board">
                <Board 
                    squares={current.squares}
                    onClick={(i) => this.handleClick(i)}
                />
            </div>
            <div className="game-info">
                <div>{status}</div>
                <ol>{moves}</ol>
            </div>
        </div>
    );
}
```



对于游戏历史中的每一步动作， 我们创建一个列表项 `<li> `，其中包含一个按钮 `<button>` 。 该按钮有一个 `onClick `处理程序，它调用一个名为 `this.jumpTo()` 的方法。 我们还没有实现 `jumpTo() `方法。 现在，我们应该看到游戏中发生的每一步动作列表，以及一条警告：

>  警告： Each child in an array or iterator should have a unique “key” prop.  Check the render method of “Game”.（数组或迭代器中的每个子元素都应该有一个唯一的 “key” prop。请检查 “Game” 的渲染方法。）

#### （3）选择一个 Key

当渲染**列表**时，React 总是存储有关每个渲染列表项的一些信息。 当更新该列表时，React 需要确定发生了什么变化。可以在列表中添加、删除、重新排列或更新项目。

`key` 是由 React 保留的特殊属性（以及 `ref` ，更高级的功能）。**创建元素时，React 将拉离 `key` 属性并将其直接存储在返回的元素上。**虽然它可能看起来像是 props(属性) 的一部分，*但是它不能通过 `this.props.key` 引用。*React 在决定哪些子元素要更新时自动使用该 `key` ；组件无法查询自己的 `key` 。

当一个列表重新渲染时， React 使用新版本中的每个元素，并在上一个列表中查找具有匹配 key(健) 的元素。当一个 key(健) 被添加到集合中时，创建一个组件；当一个 key(健) 被删除时，一个组件将会被销毁。 key(健) 用来告诉 React 关于每个组件的身份标识 ，因此他可以维持 state(状态) 到重新渲染。如果更改组件的 key(健) ，它将被完全毁灭，并重新建立一个新的 state(状态) 。

**强烈建议在构建动态列表时分配适当的 key(健) 。** 如果没有适当的值来作为 key(健) ，可能需要考虑重组你的数据。

如果没有指定任何 key(健) ，React 会报警，并回到使用数组索引作为 key(健) - 但这不是正确的选择，如果重新排序列表中的*元素* 或在列表的底部的任何位置*添加/删除项目*。明确地传递 `key={i}` 可以使警告消失，但有问题同样存在，因此在大多数情况下**不推荐**这么使用。

组件 key(健) 不需要是全局唯一的，只要相对于直系兄弟元素是唯一的就行。

#### （4）实现过程回放

在井字游戏的历史记录中，每个过去的动作都有一个与之相关的唯一ID：它是动作的连续编号。 动作永远不会重新排序，删除或从中间插入， 所以使用动作索引作为 key 是安全的。

在 Game(游戏) 组件的`render`方法中，可以将 key 添加为`<li key={move}`，并且 React 关于 key 的警告消失。

在定义 `jumpTo()` 方法前补充 `stepNumber: 0` 到 Game(游戏) 组件的 state(状态) 中，以指示我们当前正在查看的步骤。

接下来将在 Game(游戏) 组件中定义 `jumpTo` 方法来更新该 state(状态) 和 `xIsNext` 。如果步骤编号的索引为偶数，则将 `xIsNext` 设置为 `true` ：

```javascript
jumpTo(step) {
    this.setState({
        stepNumber: step,
        xIsNext: (step % 2) === 0,
    });
}
```

添加的 `stepNumber`  state(状态) 反映了现在向用户显示的动作，还需要通过添加 `stepNumber:history.length` 作为 `this.setState` 参数的一部分来更新 `stepNumber` 。 这样可以确保不会在制作新动作后显示相同的动作。

在 `handleClick(i)` 中用 `this.state.history.slice(0, this.state.stepNumber + 1)` 替换 `this.state.history` 读取。 这确保了如果我们进行了过程回放，然后从那一步开始新的动作，我们抛弃了所有后续的记录。

```javascript
handleClick(i) {
    // 创建副本，回退步数后抛弃之后的记录
    const history = this.state.history.slice(0, this.state.stepNumber + 1);
    const current = history[history.length - 1];
    const squares = current.squares.slice();

    // “胜负已分”或“方格已经有棋子”则停止响应点击事件
    if (calculateWinner(squares) || squares[i]) {
        return;
    }
    
    // 更新状态并刷新子组件
    squares[i] = this.state.xIsNext ? 'X' : 'O';
    this.setState({
        history: history.concat([{
            squares: squares,
        }]),
        // 更新步数
        stepNumber: history.length,
        // 反转状态，切换选手
        xIsNext: !this.state.xIsNext,
    });
}
```

最后，我们可以修改 Game(游戏) 组件的 `render` 方法，始终根据 `stepNumber` 渲染最后一次动作以呈现当前选定的动作：

```javascript
const current = history[this.state.stepNumber];
```

在本教程中，已经触及了许多 React 概念，包括元素，组件，props 和 state 。











