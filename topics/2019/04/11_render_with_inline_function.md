# react的render中使用内联函数的问题

## 问题描述

render里的箭头函数会使得pureComponent、shouldComponentUpdate的子组件重渲

## 问题原因

父组件向子组件传递了一个箭头函数，每次渲染都会`重新分配箭头函数`（或者使用bind的函数)。尽管子组件使用了PureComponent或者shoulComponentUpdate钩子里自定义判断，但是每次都接收到新的props，就是这个新的箭头函数，这个子组件发生重渲那是自然而然的。

## 解决思路

可以参考下面这个例子，如果要传递给子组件箭头函数，应当在父组件的render方法外面包裹一层函数再传递，这样可以保证传递给子组件的箭头函数的引用是固定不变的。

## 例子

```js
import React from 'react';
import { render } from 'react-dom';
import User from './User';

class App extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            users: [
                    { id: 1, name: 'Cory' },
                    { id: 2, name: 'Meg' },
                    { id: 3, name: 'Bob'}
                   ],
        };
    }

    deleteUser = id => {
        this.setState(prevState => {
        return {
            users: prevState.users.filter(user => user.id !== id)
            };
        });
    };

    renderUser = user => {
        // 通过这层包装，传递给组件User的onClick函数的引用就固定下来了
        return <User key={user.id} user={user} onClick={this.deleteUser} />;
    }

    render() {
        return (
            <div>
                <h1>Users</h1>
                <ul>
                    {this.state.users.map(this.renderUser)}
                </ul>
            </div>
        );
     }
}

render(<App />, document.getElementById('root'));
```

## 参考文章

1. [render函数里使用箭头函数或bind的函数造成的问题](https://medium.freecodecamp.org/why-arrow-functions-and-bind-in-reacts-render-are-problematic-f1c08b060e36)