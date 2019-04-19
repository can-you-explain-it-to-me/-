# React的Context概念

## 概念描述

React是单向数据流，Context API是解决嵌套组件中层层传递props的一种方法。通过React.createContext方法生成Context对象，该对象上挂载两个配对的组件Provider和Consumer。Provider可以嵌套。

## 如何使用

React版本需要大于`16.6`， 如果要用到useContext钩子，版本需要`16.8`以上

### 创建Context对象

```js
// 1. 创建一个Context实例, 设置默认值
const MyContext = React.createContext(defaultValue)
```

### 数据传递

```js
// 2. 使用Context的Provider组件包裹子组件
class App extends React.PureComponent {
    render() {
        return (
            <MyContext.Provider value="black">
                <ChildComp></ChildComp>
            </MyContext.Provider>
        )
    }
}
```

### 子组件获Context的值

有三种方式：

#### 1. 子组件为类组件

两步操作：第一将Context实例赋值给class的contextType属性， 第二class组件中就可以通过this.context拿到Context的value值。该方法为类组件特有方法。

```js
class ChildComp extends PureComponent {
    // contextType为约定的类的静态属性名称
    static contextType = MyContext
    render() {
        // this.context也是约定的
        const val = this.context
        // 类组件也可以借助Consumer组件来获取Context数据
        return <span>{val}</span>
    }
}
```

#### 2. 子组件为函数组件

像函数组件，没有类的静态属性，无法像上面那样的操作。这时候可以用上Consumer组件了, Consumer组件内部使用render props

```js
function ChildComp() {
    return (
        // 这个可以提取到任意层级，只要在对应Provider内部就行
        <MyContext.Consumer>
            {val => <span>{val}</span>}
        </MyContext.Consumer>
    )
}
```

#### 3. 使用useContext()钩子

```js
function ChildComp() {
    // 传入生成的Context对象
    const context = React.useContext(MyContext)
    // 使用
    return <span>{context.value}</span>
}
```

## 代码实例

### 示例1：换肤功能

[![Edit Context示例](https://codesandbox.io/static/img/play-codesandbox.svg)](https://codesandbox.io/s/lpw2jjl6nl?fontsize=14)

## 参考文章

1. [React官网Context](<https://reactjs.org/docs/context.html>)
2. [Everything you need to know about React’s Context API](<https://hackernoon.com/everything-you-need-to-know-about-reacts-context-api-e5c8c32ef202>)
3. [Using Context in React](<https://medium.com/@wisecobbler/using-context-in-react-56a8e7da5431>) 比较贴近官方的介绍，在业务数据模块使用Context

## 