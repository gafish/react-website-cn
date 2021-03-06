[文档](/cn/docs/hello-world.md) | [教程](/cn/tutorial/tutorial.md) | [社区](/cn/community/support.md) | [博客](/cn/_posts/2017-04-07-react-v15.5.0.md) | [React Github](https://facebook.github.io/react/)

---
<details>
  <summary>导航</summary>

#### 快速入门

* [安装](/cn/docs/installation.md)
* [Hello World](/cn/docs/hello-world.md)
* [JSX 介绍](/cn/docs/introducing-jsx.md)
* [渲染元素](/cn/docs/rendering-elements.md)
* [组件和Props](/cn/docs/components-and-props.md)
* [State和生命周期](/cn/docs/state-and-lifecycle.md)
* [事件处理](/cn/docs/handling-events.md)
* [条件渲染](/cn/docs/conditional-rendering.md)
* [列表和键](/cn/docs/lists-and-keys.md)
* [表单](/cn/docs/forms.md)
* [状态提升](/cn/docs/lifting-state-up.md)
* [组合 vs 继承](/cn/docs/composition-vs-inheritance.md)
* [用 React 思考](/cn/docs/thinking-in-react.md)

#### 高级教程

* [深入JSX](/cn/docs/jsx-in-depth.md)
* [使用 PropTypes 做类型检查](/cn/docs/typechecking-with-proptypes.md)
* [Refs 和 DOM](/cn/docs/refs-and-the-dom.md)
* [不可控组件](/cn/docs/uncontrolled-components.md)
* [性能优化](/cn/docs/optimizing-performance.md)
* [不使用 ES6 的 React](/cn/docs/react-without-es6.md)
* [不使用 JSX 的 React](/cn/docs/react-without-jsx.md)
* [一致性比较（Reconciliation）](/cn/docs/reconciliation.md)
* [上下文（Context）](/cn/docs/context.md)
* [Web Components](/cn/docs/web-components.md)
* [高阶组件](/cn/docs/higher-order-components.md)
* [**`与其它类库集成`**](/cn/docs/integrating-with-other-libraries.md)

#### 参考

* [React API](/cn/docs/react-api.md)
* [React.Component](/cn/docs/react-component.md)
* [ReactDOM](/cn/docs/react-dom.md)
* [ReactDOMServer](/cn/docs/react-dom-server.md)
* [DOM 元素](/cn/docs/dom-elements.md)
* [合成事件（SyntheticEvent）](/cn/docs/events.md)

#### 贡献

* [如何贡献](/cn/contributing/how-to-contribute.md)
* [代码库概述](/cn/contributing/codebase-overview.md)
* [实现说明](/cn/contributing/implementation-notes.md)
* [设计原则](/cn/contributing/design-principles.md)


</details>

# 与其它类库集成

React 可以在任何web应用中使用。它能嵌入其它的应用，并且只要小心一点，其它的应用也能嵌入到 React。本指南将检查一些更常见的用例，重点是 [jQuery](https://jquery.com/) 和 [Backbone](http://backbonejs.org/) 的集成，但是同样的想法也能应用于将组件和任何现有代码集成。

## 与 DOM 操作插件集成

React 不知道在 React 以外对 DOM 的修改。它决定更新是基于它自己的内部的表现，并且如果同样的 DOM 节点被其它库操作，React 会感到困惑并且没有办法恢复。

这并不意味着将React与影响DOM的其他方式相结合是不可能的，甚至是一定难以理解的，你只需要注意每个人在做什么。

避免冲突最简单的方式是防止 React 组件更新。您可以通过渲染React的元素来完成此操作，无需更新，像一个空的  `<div />`.

### 如何解决问题

为了演示这个，我们绘制一个包裹通用 jQuery 插件的容器。

我们将附加一个[ref](/cn/docs/refs-and-the-dom.md) 到根 DOM 节点。在 `componentDidMount` 里，我们将获取一个到它的引用，以便我们能将它传递到 jQuery 插件。

为了防止 React 在挂载之后接触 DOM，我们将从 `render()` 方法中返回一个空的 `<div />` 。这个 `<div />` 元素没有属性或子元素，因此 React 无须更新它，让 jQuery 插件可以自由的管理 DOM 的一部分：

```javascript
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

注意我们定义了 `componentDidMount` 和 `componentWillUnmount` 两个[生命周期钩子](/cn/docs/react-component.md#the-component-lifecycle). 一些 jQuery 插件附加 DOM 事件监听，因此在 `componentWillUnmount` 中分离它们是非常重要的。如果插件没有指定清除的方法，你或许不得不自己提供，记住移除插件注册的任何事件监听防止内存泄漏。

### 与 jQuery 选择插件集成

对于这些概念的一个更具体的例子，让我们为插件[Chosen]（https://harvesthq.github.io/chosen/）写一个最小的包装，这增加了`<select>`的输入。

>**注意:**
>
>只是因为这是可能的，并不意味着它是React应用程序的最佳方法。如果可以的话，我们鼓励你使用React组件。 React 组件在 React 应用程序中更容易重用，并且通常可以更好地控制其行为和外观。

首先，让我们看看 Chosen 对 DOM 做了什么

如果你在 `<select>` DOM节点上调用它，它读取原始 DOM 节点的属性，通过一个内联样式隐藏它，然后附加一个带有自身视觉表现的独立 DOM 节点在 `<select>` 后面。然后它触发 jQuery 事件去通知我们有关变动。

假设这是我们正在使用的 `<Chosen>` 包装器 React 组件的API。

```javascript
function Example() {
  return (
    <Chosen onChange={value => console.log(value)}>
      <option>vanilla</option>
      <option>chocolate</option>
      <option>strawberry</option>
    </Chosen>
  );
}
```

为简单起见，我们将作为一个 [非受控组件](/react/docs/uncontrolled-components.html) 来实现。

首先，我们将创建一个带有 `render()` 方法的空组件，它返回一个包裹在 `<div>` 里的`<select>`：

```javascript
class Chosen extends React.Component {
  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

注意我们如何将 `<select>` 包裹到一个额外的 `<div>`，这是必要的，因为 Chosen 将在传递给它的 `<select>` 节点后面附加另外的 DOM 元素。然而，就 React 而言，`<div>` 经常只有单个子节点。这是我们如何确保 React 更新将不会和额外的 DOM 节点冲突。重要的是，如果你在 React 外部修改 DOM 时，你必须确保 React 没有理由接触到 DOM 节点。

接下来，我们将实现生命周期钩子。我们需要使用'componentDidMount`中的`<select>`节点的引用来初始化Chosen，并且在 `componentWillUnmount` 里销毁它:

```javascript
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();
}

componentWillUnmount() {
  this.$el.chosen('destroy');
}
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/qmqeQx?editors=0010)

注意 React 对 `this.el` 字段没有特别的含义。它只是可以工作，因为我们在 `render()` 方法里已经预先把 `ref` 赋值给了这个字段；

```js
<select className="Chosen-select" ref={el => this.el = el}>
```

这足够让我们的组件渲染，但是我希望得到关于值变化的通知。为此，我们将在通过 Chosen 管理的 `<select>` 上订阅 jQuery `change` 事件。

我们将不会直接传递 `this.props.onChange` 给 Chosen，因为组件的属性（props）可能随时会改变，并且包含事件处理器。相反，我们将声明一个 `handleChange()` 方法去调用 `this.props.onChange`，并且将其订阅到 jQuery `change` 事件：

```javascript
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();

  this.handleChange = this.handleChange.bind(this);
  this.$el.on('change', this.handleChange);
}

componentWillUnmount() {
  this.$el.off('change', this.handleChange);
  this.$el.chosen('destroy');
}

handleChange(e) {
  this.props.onChange(e.target.value);
}
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/bWgbeE?editors=0010)

最后，还有一件事情要做，在 React 里，属性（props）随时会变化。例如，假如父组件的状态（state）变化时， `<Chosen>` 组件能获取不同的子节点。这意味着在集成点上，重要的是我们手动更新 DOM 在响应更新，直到我们不再让 React 为我们管理 DOM。

Chosen 文档建议我们能使用 jQuery `trigger()` API 来通知它有关原始 DOM 元素的变动。我们将让 React 注意更新 `<select>` 内部的 `this.props.children` ，但是我们也将添加一个 `componentDidUpdate()` 生命周期钩子来通知 Chosen 关于子节点列表的变化：

```javascript
componentDidUpdate(prevProps) {
  if (prevProps.children !== this.props.children) {
    this.$el.trigger("chosen:updated");
  }
}
```

这样，当 React 更改管理的 `<select>` 子节点时，Chosen 将知道更新它的 DOM 元素。

`Chosen` 组件完整的实现看起来像这样：

```javascript
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }

  componentDidUpdate(prevProps) {
    if (prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }

  handleChange(e) {
    this.props.onChange(e.target.value);
  }

  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/xdgKOz?editors=0010)

## 与其他视图（View）库集成

React 能被嵌入到其它的应用，这多亏了灵活的[`ReactDOM.render()`](/cn/docs/react-dom.md#render).

虽然 React 在启动时通常用于将单个根 React 组件加载到DOM中，但对于UI的独立部分，`ReactDOM.render()` 也能被调用多次，可以像按钮一样小，或者像一个应用程序那么大。

事实上，这恰好是如何在 Facebook 中使用 React。这让我们在 React 中逐个编写应用程序，并将其与我们现有的服务器生成的模板和其他客户端代码相结合。

### 用 React 替换基于字符串的渲染

旧的web应用的一个常见模式是将DOM块作为一个字符串来描述，并且像 `$el.html(htmlString)` 这样把它插入到DOM。代码库的这些要点可以完美的引入到 React。只要将基于字符串的渲染作为一个 React 组件重写。

因此，下面的 jQuery 实现...

```javascript
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```

...可以被作为 React 组件重写:

```javascript
function Button() {
  return <button id="btn">Say Hello</button>;
}

ReactDOM.render(
  <Button />,
  document.getElementById('container'),
  function() {
    $('#btn').click(function() {
      alert('Hello!');
    });
  }
);
```

从这里你可以开始移动更多的逻辑到组件，并且采用更多常见的 React 实践。例如，在组件里最好不要依赖 ID，因为同一个组件会被多次渲染。相反，我们将使用 [React 事件系统](/cn/docs/handling-events.md) 并且在 React  `<button>` 元素上注册点击处理程序：

```javascript
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/RVKbvW?editors=1010)

你可以拥有许多像这样你喜欢的独立组件，并使用`ReactDOM.render()` 将它们渲染到不同的 DOM 容器。渐渐的，当你将更多的应用转换到 React，你将能够将它们组合到更大的组件，并将一些 `ReactDOM.render()` 调用移动到层次结构中。

### 在 Backbone 视图中嵌入 React

[Backbone](http://backbonejs.org/) 视图典型的用法是使用 HTML 字符串或字符串生成模板函数，为他们的 DOM 元素创建内容，这个过程也能被渲染 React 组件替换。

下面我们将创建一个名为 `ParagraphView` 的Backbone视图。 它将覆盖Backbone的 `render()` 函数，以将React `<Paragraph>` 组件渲染到由Backbone（`this.el`）提供的DOM元素中。 这里也是使用 [`ReactDOM.render()`](/cn/docs/react-dom.md#render):

```javascript
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extend({
  render() {
    const text = this.model.get('text');
    ReactDOM.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/gWgOYL?editors=0010)

在 `remove` 方法里也调用 `ReactDOM.unmountComponentAtNode()` 是非常重要的，因此当它被销毁时 React 移除事件处理器和组件树其它关联的资源。

当组件从 React 树上移除时，会自动执行清除，但是因为我们是通过手工方式移除这个入口，我们必须调用这个方法。

## 与模型层(Model Layers)集成

虽然一般来说推荐使用单向数据流，比如 [React state](/cn/docs/lifting-state-up.md), [Flux](http://facebook.github.io/flux/), or [Redux](http://redux.js.org/), 但是 React 组件也能使用其它框架和库的模型层。

### 在 React 组件中使用 Backbone Models

在 React 组件里使用 [Backbone](http://backbonejs.org/) 模型(Models)和集合(Collections)的最简单的方法是监听 change 事件并手动强制更新。

负责渲染模型(Models)的组件将监听 `'change'` 事件，而负责渲染集合(Collections)的组件将监听 `'add'` 和 `'remove'` 事件。这两种情况里，调用 [`this.forceUpdate()`](/cn/docs/react-component.md#forceupdate) 以使用新数据重新渲染组件。

在下面的示例中，`List` 组件渲染 Backbone 集合(Collections)，使用 `Item` 组件渲染单个的项目。

```javascript
class Item extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.model.on('change', this.handleChange);
  }

  componentWillUnmount() {
    this.props.model.off('change', this.handleChange);
  }

  render() {
    return <li>{this.props.model.get('text')}</li>;
  }
}

class List extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange();
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.collection.on('add', 'remove', this.handleChange);
  }

  componentWillUnmount() {
    this.props.collection.off('add', 'remove', this.handleChange);
  }

  render() {
    return (
      <ul>
        {this.props.collection.map(model => (
          <Item key={model.cid} model={model} />
        ))}
      </ul>
    );
  }
}
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/GmrREm?editors=0010)

### 从 Backbone Models 中提取数据

上述方法需要你的 React 组件知道 Backbone 模型和集合。如果你以后计划迁移到其它数据管理方案，你肯定不愿意在 Backbone 这里改动太多的代码。

一个解决方案是当它改变时提取模型的属性作为纯数据，并且将这个逻辑保留在一个位置。以下是 [一个高阶组件](/cn/docs/higher-order-components.md) 用于提取 Backbone 模型所有的属性到状态(state)，传递数据到包裹组件。

这种方式，只有高阶组件需要知道关于 Backbone 模型的内部构造，并且在应用中的大多数组件可以与 Backbone 保持不变。

在下面的示例里，我们将复制模型的属性以形成初始状态。我们订阅 `change` 事件 (在 unmounting 中取消订阅),当它发生变化时，我们将通过模型的当前属性更新状态。最终，我们确保如果 `model` 属性自己变化时，不要忘了从以前的模型里取消订阅，并在新的模型里订阅。

请注意，此示例并不意味着与 Backbone 工作有关的细节，但它应该为您提供如何以通用方式处理这个问题的想法：

```javascript
function connectToBackboneModel(WrappedComponent) {
  return class BackboneComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = Object.assign({}, props.model.attributes);
      this.handleChange = this.handleChange.bind(this);
    }

    componentDidMount() {
      this.props.model.on('change', this.handleChange);
    }

    componentWillReceiveProps(nextProps) {
      this.setState(Object.assign({}, nextProps.model.attributes));
      if (nextProps.model !== this.props.model) {
        this.props.model.off('change', this.handleChange);
        nextProps.model.on('change', this.handleChange);
      }
    }

    componentWillUnmount() {
      this.props.model.off('change', this.handleChange);
    }

    handleChange(model) {
      this.setState(model.changedAttributes());
    }

    render() {
      const propsExceptModel = Object.assign({}, this.props);
      delete propsExceptModel.model;
      return <WrappedComponent {...propsExceptModel} {...this.state} />;
    }
  }
}
```

为了展示如何使用它，我们将连接一个 `NameInput` React 组件到 Backbone 模型，并且在每次输入改变时更新它的 `firstName` 属性：

```javascript
function NameInput(props) {
  return (
    <p>
      <input value={props.firstName} onChange={props.handleChange} />
      <br />
      My name is {props.firstName}.
    </p>
  );
}

const BackboneNameInput = connectToBackboneModel(NameInput);

function Example(props) {
  function handleChange(e) {
    model.set('firstName', e.target.value);
  }

  return (
    <BackboneNameInput
      model={props.model}
      handleChange={handleChange}
    />
  );
}

const model = new Backbone.Model({ firstName: 'Frodo' });
ReactDOM.render(
  <Example model={model} />,
  document.getElementById('root')
);
```

[在 CodePen 中试试](http://codepen.io/gaearon/pen/PmWwwa?editors=0010)

这种技术不限于 Backbone。 您可以通过在生命周期钩子里订阅其更改，并可选地将数据复制到本地 React 状态，将 React 与任何模型库配合使用。


---

* 上一篇：[高阶组件](/cn/docs/higher-order-components.md)
* 下一篇：[React API](/cn/docs/react-api.md)
