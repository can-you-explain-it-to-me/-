# react 代码复用的几种方式

##问题描述

`react`提供的几种代码复用方式

## 问题原因

无

## 解决思路

1. `mixins`

可以解决横切关注点([`cross-cutting concerns`](https://en.wikipedia.org/wiki/Cross-cutting_concern))问题。
> 如果一个组件有多个混入，且其中几个混入中定义了相同的生命周期方法（比如都会在组件被摧毁的时候执行），那么这些生命周期方法是一定会被调用的。通过混入定义的方法，执行顺序也与定义时的顺序一致，且会在组件上的方法执行之后再执行。

这种方法不支持`ES6`写法，且`React`官方也不建议使用。`Dan`也不建议(具体缺点见参考文章4)

### 例子

```js
// 1.mixins
var SetIntervalMixin = {
  componentWillMount: function() {
    this.intervals = [];
  },
  setInterval: function() {
    this.intervals.push(setInterval.apply(null, arguments));
  },
  componentWillUnmount: function() {
    this.intervals.forEach(clearInterval);
  }
};

var createReactClass = require('create-react-class');

// only supported in createReactClass
var TickTock = createReactClass({
  mixins: [SetIntervalMixin], // Use the mixin
  getInitialState: function() {
    return {seconds: 0};
  },
  componentDidMount: function() {
    this.setInterval(this.tick, 1000); // Call a method on the mixin
  },
  tick: function() {
    this.setState({seconds: this.state.seconds + 1});
  },
  render: function() {
    return (
      <p>
        React has been running for {this.state.seconds} seconds.
      </p>
    );
  }
});

ReactDOM.render(
  <TickTock />,
  document.getElementById('example')
);
```

2. `render props`

同样可以解决横切关注点([`cross-cutting concerns`](https://en.wikipedia.org/wiki/Cross-cutting_concern))问题。
> 简单来说就是提供一个带有 **函数`prop`** 的组件，它能够动态决定什么需要渲染的，而不是使用硬编码来有效地改变它的渲染结果。<br>
更具体地说，`render props`是一个组件用来了解要渲染什么内容的 **函数`prop`**。

能用`HOC`处理的理论上也可以用`render props`实现，反过来也一样。

### 例子

```js
//2.render props
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.handleMouseMove = this.handleMouseMove.bind(this);
    this.state = { x: 0, y: 0 };
  }

  handleMouseMove(event) {
    this.setState({
      x: event.clientX,
      y: event.clientY
    });
  }

  render() {
    return (
      <div style={{ height: '100%' }} onMouseMove={this.handleMouseMove}>

        {/*
          Instead of providing a static representation of what <Mouse> renders,
          use the `render` prop to dynamically determine what to render.
        */}
        {this.props.render(this.state)}
      </div>
    );
  }
}

class MouseTracker extends React.Component {
  constructor(props) {
    super(props);

    // This binding ensures that `this.renderTheCat` always refers
    // to the *same* function when we use it in render.
    this.renderTheCat = this.renderTheCat.bind(this);
  }

  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        {/*
          deliver a render method to component Mouse.
        */}
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

3. `HOC`

`mixins`的升级版。依然可以解决横切关注点([`cross-cutting concerns`](https://en.wikipedia.org/wiki/Cross-cutting_concern))问题。
> 高阶组件就是一个函数，且该函数接受一个组件作为参数，并返回一个新的组件。

该函数应该是一个没有副作用(`side effect`)的纯函数。它增强了原组件的功能，但并不改变其原有的属性。

### 例子

```js
//3.HOC
// This function takes a component...
function withSubscription(WrappedComponent, selectData) {
  // ...and returns another component...
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.handleChange = this.handleChange.bind(this);
      this.state = {
        data: selectData(DataSource, props)
      };
    }

    componentDidMount() {
      // ... that takes care of the subscription...
      DataSource.addChangeListener(this.handleChange);
    }

    componentWillUnmount() {
      DataSource.removeChangeListener(this.handleChange);
    }

    handleChange() {
      this.setState({
        data: selectData(DataSource, this.props)
      });
    }

    render() {
      // ... and renders the wrapped component with the fresh data!
      // Notice that we pass through any additional props
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

## 参考文章

1. [mixins](https://reactjs.org/docs/react-without-es6.html)
2. [render props](https://reactjs.org/docs/render-props.html)
3. [HOC](https://reactjs.org/docs/higher-order-components.html)
4. [Mixins Are Dead. Long Live Composition](https://medium.com/@dan_abramov/mixins-are-dead-long-live-higher-order-components-94a0d2f9e750)
