# 为什么要用React Hooks？

前言：Hooks在React16.8上登场，主要是解决组件逻辑复用和class组件存在的问题。Hooks只能在函数组件中使用，意味着class组件不能使用它，React团队也希望Hooks能慢慢替换掉class组件，以后新的组件可以使用Hooks来编写。当然React也不打算去掉class组件，毕竟还是不少人在用。

## Class组件的现状

现在我们用到的组件基本上都是class组件，但class组件存在以下缺点：

 1. 组件之间很难复用状态逻辑；

   虽然已经有一些解决方案存在了，例如：render props和高阶组件，但它们都需要你重新组织组件，而且会使你的组件变的笨重和难以理解，更蛋疼的地方是会把你的组件嵌套越来越深。

 2. 复杂组件变得难以理解；

   举个例子，我们需要在componentDidMount和componentDidUpdate上进行拉取数据，但我们也会在componentDidMount里面注册和非拉取数据逻辑相关的事件监听，然后在componentWillUnmount清除事件监听。生命周期里面有太多逻辑混杂在一起。
 3. Class中的this容易让人疑惑；
   this的问题很多人都遇到过，在render方法里面引入函数我们还要先bind一下。

## Hooks的思想

什么是Hooks？Hooks是一个React状态和生命周期的钩子。Hooks可以很好的解决Class组件存在的问题。我们不再需要记各种生命周期方法，直接使用useEffect就行了，使用Hooks后我们也不再需要this了。
使用Hooks有一些潜规则，在使用之前需要我们先了解，不然容易一脸疑惑。

### 只在函数组件最顶层调用Hooks

不要在循环语句、条件语句或者嵌套函数内调用Hooks。这是为了确保在组件渲染时Hooks被调用的顺序是一致的。

### 只在React函数组件内使用Hooks

不要在正常JS函数里使用Hooks，但你可以在React函数组件和自定义Hooks里面调用Hooks。

## React内置Hooks介绍

在使用Hooks之前，我们安装一下eslint-plugin-react-hooks，它可以帮助我们提醒和修复Hooks的问题。

``` console
npm install eslint-plugin-react-hooks -D
```

ESLint配置：

``` json
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

 1. useState
   useState是用来申明组件状态变量的Hook，等价于Class组件的this.state。
   Class组件申明状态：

``` javascript
class Example extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      count: 0
    };
  }

  onBtnClick = () => {
      this.setState({
          count: ++this.state.count
      });
  }

  render() {
      return <div><span>{this.state.count}</span><button onClick={this.onBtnClick}>按钮</button></div>;
  }
}
```

   使用useState申明状态：

``` javascript
import React, { useState, useCallback } from 'react';

function Example() {
    const [count, setCount] = useState(0);
    const btnClickHandler = useCallback(() => {
        setCount((count) => {
            return count + 1;
        });
    }, []);


    return <div><span>{count}</span><button onClick={btnClickHandler}>按钮</button></div>
}
```

useCallback主要是解决re-render的问题，后面会介绍到。
我们可以看到useState返回一个数组，数组第一个是可读状态，第二个参数是设置状态，我们可以看到useState返回一个数组，数组第一个是可读状态，第二个参数是设置状态。的参数传入的是状态的初始值。
如果我们想要用多个状态可以使用多个useState，可以使用两种方式：

``` javascript
// 第一种
function ExampleWithManyStates() {
  // Declare multiple state variables!
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);

  // ...
}

// 第二种
function ExampleWithManyStates2() {
  // Declare multiple state variables!
  const [state, setState] = useState({
      age: 42,
      fruit: 'banana',
      todos: [{ text: 'Learn Hooks' }]
  });

  // ...

  // setState example
  const handler = useCallback(() => {
    setState((state) => {
        return { ...state, age: 18 };
    });
  }, []);
  // ...
}
```

 2. useEffect

 3. useContext

 4. useReducer

 5. useCallback

 6. useMemo

 7. useRef

 8. useImperativeHandle

 9. useLayoutEffect

 10. useDebugValue

## Hooks与TypeScript

## Hooks实践
