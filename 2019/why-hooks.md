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

### 什么是Hooks？

Hooks是一个React状态和生命周期的钩子。Hooks可以很好的解决Class组件存在的问题。我们不再需要记各种生命周期方法，直接使用useEffect就行了，使用Hooks后我们也不再需要this了。
使用Hooks有一些潜规则，在使用之前需要我们先了解，不然容易一脸疑惑。

#### 只在函数组件最顶层调用Hooks

不要在循环语句、条件语句或者嵌套函数内调用Hooks。这是为了确保在组件渲染时Hooks被调用的顺序是一致的。

#### 只在React函数组件内使用Hooks

不要在正常JS函数里使用Hooks，但你可以在React函数组件和自定义Hooks里面调用Hooks。

### Hooks的运行机制

Hooks是怎么知道状态对应哪个useState？答案是依赖于Hooks的调用顺序。

``` javascript
function Form() {
    // 1. Use the name state variable
    const [name, setName] = useState('Mary');

    // 2. Use an effect for persisting the form
    useEffect(function persistForm() {
        localStorage.setItem('formData', name);
    });

    // 3. Use the surname state variable
    const [surname, setSurname] = useState('Poppins');

    // 4. Use an effect for updating the title
    useEffect(function updateTitle() {
        document.title = name + ' ' + surname;
    });

    // ...
}
```

上面的代码每次渲染顺序是：

``` javascript
// ------------
// First render
// ------------
useState('Mary')           // 1. Initialize the name state variable with 'Mary'
useEffect(persistForm)     // 2. Add an effect for persisting the form
useState('Poppins')        // 3. Initialize the surname state variable with 'Poppins'
useEffect(updateTitle)     // 4. Add an effect for updating the title

// -------------
// Second render
// -------------
useState('Mary')           // 1. Read the name state variable (argument is ignored)
useEffect(persistForm)     // 2. Replace the effect for persisting the form
useState('Poppins')        // 3. Read the surname state variable (argument is ignored)
useEffect(updateTitle)     // 4. Replace the effect for updating the title

// ...
```

只要每次渲染Hooks的调用顺序一样，React就可以把它们和局部状态联系起来。
如果我们想要把Hook放在条件语句里就会导致出问题，这就是为什么我们只在函数组件最顶层调用Hooks。如果我们想要有条件判断地运行Hooks，可以把条件语句放在Hooks里面：

``` javascript
useEffect(function persistForm() {
    // 👍 We're not breaking the first rule anymore
    if (name !== '') {
        localStorage.setItem('formData', name);
    }
});
```

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

- useState

    ``` javascript
    const [state, setState] = useState(initialState);
    ```

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

### 函数式更新

setState还支持传入函数来更新状态，如果新值的状态需要使用到旧值就可以用函数式更新。

``` javascript
function Counter({initialCount}) {
    const [count, setCount] = useState(initialCount);
    return (
        <>
        Count: {count}
        <button onClick={() => setCount(initialCount)}>Reset</button>
        <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
        <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
        </>
    );
}
```

注意：useState不会自动合并更新对象，你可以使用对象扩展语法对对象进行更新。

``` javascript
setState(prevState => {
    // Object.assign would also work
    return {...prevState, ...updatedValues};
});
```

### 懒初始化

useState的初始化值还可以是函数，可以把一些昂贵的操作放在函数里，这样只有第一次渲染会执行，后面就不会执行了。

``` javascript
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

- useEffect

- useContext

- useReducer

- useCallback

- useMemo

- useRef

- useImperativeHandle

- useLayoutEffect

- useDebugValue

## Hooks与TypeScript

## Hooks实践
