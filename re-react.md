# 重读react api
## 顶层api React > 组件 
### React.PureComponent  
```
通过子类React.Component 和 React.PureComponent定义react组件 
``` 
PureComponent与component的区别在于，PureComponent实现了shouldComponentUpdate()方法，通过浅层对比prop和state的方式来优化性能

### React.memo  
高阶组件,通过记忆组件渲染结果的方式来提高组件的性能表现  
与React.PureComponent相似，但是只适用于函数组件，不适用于class组件

```
const MyComponent = React.memo(function MyComponent(props) {
  /* 使用 props 渲染 */
});
```
函数组件在给定相同 props 的情况下渲染相同的结果，那么可以将其包装在React.memo 中调用，React 将跳过渲染组件的操作并直接复用最近一次渲染的结果  
默认浅层对比，可以自定义比较函数作为第二个参数传入react.memo()

**React.PureComponent和React.memo都是作为性能优化的方式存在，并非用来阻止渲染**

## 顶层api React > createElemrnt()
JSX 语法会调用createElemrnt(),使用 JSX 编写的代码将会被转换成使用 React.createElement() 的形式
```
React.createElement(
  type,
  [props],
  [...children]
)
```

## 顶层api React > 操作元素API

