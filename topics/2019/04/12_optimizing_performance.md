# React性能优化的方法

web应用的优化角度和方法很多，这里单单讲单页应用和React自带的性能优化方案。

## 方法1：代码分割

> 这个方法对单页应用框架都适用，该方法基于单页应用通过打包工具将应用的多个模块`打包`成一个文件。

#### 问题：

随着应用逐渐丰富或者引入第三方包时，打包成一个文件会造成打包的文件变大，最终影响应用首屏加载耗时延长，影响体验。终极目标是实现"按需加载"。

#### 实现方式：

如果是create-react-app创建的项目，项目自动支持代码分割技术。下面讲自己使用webpack配置项目实现代码分割。

这一部分内容参考[webpack关于Code Splitting的介绍](<https://webpack.js.org/guides/code-splitting/>)。这一章主要讲了3种代码分割的方式：

1. `通过entry属性，设置多个入口点`，生成多个打包文件，自己按需引入到项目中。这里需要借助[SplitChunkPlugin](https://webpack.js.org/plugins/split-chunks-plugin/)库，在配置文件中设置optimization属性配置，来去除多个模块中重复引用同一个包的情况

```javascript
optimization: {
		splitChunks: {
		chunks: 'all'
	}
}
```

2. 使用 [import()](https://webpack.js.org/api/module-methods#import/) 动态加载模块，注意是import()方法，不是es6的块导入方法`import X from "module"`。这个才是真正的按需加载，只有用户来到这个模块，模块才会被加载。import()方法返回的是一个promise，因此通过then方法链式操作，或者async、await操作都可以。`import('path/to/module') -> Promise`

```js
async function getComponent () {
  var element = document.createElement('div');
	/**
	* 注意两点： 
	* 1、import()方法的参数，bundlename, 接收你引入的包名
	* 2、注意这里面的注释，webpackChunkName: "lodash" ,这个是有意义的，webpackChunkName的值将会使独立包名被设置为lodash.bundle.js
	* ，如果你不写这个注释，那输出的独立包将会是[id].bundle.js。这里增加了可读性
	*/
  const { default: _ } = await import(/* webpackChunkName: "lodash" */ 'lodash');
  element.innerHTML = _.join(['Hello', 'webpack'], ' ');
  return element;
}
// async函数返回一个Promise对象
getComponent().then(component => {
  document.body.appendChild(component);
});
```

3. Webpack 4.6新增了prefetching和preload两大模块。(待学习补充...)

## 参考文章

1. [React16加载性能优化指南--掘金](<https://juejin.im/post/5b506ae0e51d45191a0d4ec9#heading-0>)
2. [React官网高阶用法--代码切割技术](<https://reactjs.org/docs/code-splitting.html>)