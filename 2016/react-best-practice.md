#React的一些最佳实践

以下是我使用了一年多的React总结的经验：

###1.建议使用ES6的模块系统
``` javascript
import React, { Component, PropTypes } from 'react'
import { connect } from 'react-redux'

```
为什么呢？
因为使用ES6的模块的代码可被静态分析，例如Rollup和Webpack2的Tree Shaking就会通过静态分析，去除不必要的代码。

###2.组件声明必须要有默认属性和属性类型
帮助其他人和自己使用组件
```javascript
Tabs.defaultProps = {
    onSelect: null,
    activeLinkStyle: null,
    defaultSelectedTab: '',
    className: '',
    style: null
};
Tabs.propTypes = {
    onSelect: PropTypes.func,
    activeLinkStyle: PropTypes.object,
    defaultSelectedTab: PropTypes.string,
    className: PropTypes.string,
    style: PropTypes.object
};
```

###3.使用ES6的类来声明React组件
```javascript
export class Test extends Component {
    constructor(props, context){
        super(props, context);
        
        this.onClick = this.onClick.bind(this);
    }
    
    componentWillmount(){}
    
    componentDidMount(){}
    
    componentWillReceiveProps(){}
    
    shouldComponentUpdate(){}
    
    componentWillUpdate(){}
    
    componentDidUpdate(){}
    
    componentWillUnmount(){}
    
    render(){
        return (
            <div onClick={ this.onClick }>test</div>
        )
    }
}
````
使用ES6的类声明React组件的时候需要注意绑定事件的时候需要bind一下this。
使用ES6的类声明组件的好处是，我们可以通过class继承来复用基类组件，而官方已经不推荐使用mixins了。

###4.shouldComponentUpdate优化
React默认的shouldComponentUpdate()的值是返回true，这导致组件会重复render，和做diff计算。我们需要对组件的shouldComponentUpdate做优化，相同props和state就不渲染。
```javascript
var hasOwnProperty = Object.prototype.hasOwnProperty;

function is(x, y) {
    if (x === y) {
        return x !== 0 || 1 / x === 1 / y;
    } else {
        if(typeof x === 'function' && typeof y === 'function'){
            return x.toString() === y.toString();
        }
        return x !== x && y !== y;
    }
}

function shallowEqual(objA, objB) {
    if (is(objA, objB)) {
        return true;
    }

    if (typeof objA !== 'object' || objA === null || typeof objB !== 'object' || objB === null) {
        return false;
    }

    var keysA = Object.keys(objA);
    var keysB = Object.keys(objB);

    if (keysA.length !== keysB.length) {
        return false;
    }

    for (var i = 0; i < keysA.length; i++) {
        if (!hasOwnProperty.call(objB, keysA[i]) || !is(objA[keysA[i]], objB[keysA[i]])) {
            return false;
        }
    }

    return true;
}


export class Test extends Component {
    constructor(props, context){
        super(props, context);
        
        this.onClick = this.onClick.bind(this);
    }
    
    componentWillmount(){}
    
    componentDidMount(){}
    
    componentWillReceiveProps(){}
    
    shouldComponentUpdate(nextProps, nextState){
            let shouldUpdate = !shallowEqual(this.props, nextProps) || !shallowEqual(this.state, nextState);
    
            if(shouldUpdate && process.env.NODE_ENV !== 'production') {
                console.log('Component: ' + this.constructor.name + ' will update');
            }
    
            return shouldUpdate;
        }
    
    componentWillUpdate(){}
    
    componentDidUpdate(){}
    
    componentWillUnmount(){}
    
    render(){
        return (
            <div onClick={ this.onClick }>test</div>
        )
    }
}
````
上面代码对新旧props和新旧state比对，shallowEqual用的React的shallowEqual，但我对它做了个小修改，让它支持对函数的对比，使用的是toString()后的比对

###5.Container和Presentational组件划分
顶层组件，例如页面组件使用Container，因为通常涉及状态变更；Presentational组件一般是普通UI组件，无状态或者很少状态。

###6.一个全局store，组件私有状态可通过中介者通信，方便解耦
![communicate](./imges/store.png)

###7.工具相关

###8.同构渲染相关

###9构建相关

