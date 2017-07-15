## 前言
我们平常写vue的组件时，一般都是用的是模版，这种方式看起来比较简洁，而且vue作者也推荐使用这个方式，但是这种方式也有一些它的弊端，例如模版调试麻烦，或者在一些场景下模版描述可能没那么简单和方便。

下面我们要讲的是如何在vue里面写jsx，知道react的人应该都知道jsx，jsx的一个特性就是非常灵活，虽然有的人觉得jsx很丑陋，把逻辑都写到模版的感觉，但萝卜青菜各有所爱，适合自己适合团队的就是最好的。

<!--more-->

在使用jsx之前我们需要安装一个babel插件（babel-plugin-transform-vue-jsx ）
安装方式：
```
npm install\
  babel-plugin-syntax-jsx\
  babel-plugin-transform-vue-jsx\
  babel-helper-vue-jsx-merge-props\
  babel-preset-es2015\
  --save-dev
```
然后再.babelrc里面添加：
``` json
{
  "presets": ["es2015"],
  "plugins": ["transform-vue-jsx"]
}
```
接着我们就可以愉快地在vue里面编写jsx了。
Test.vue
``` html
<script>
<script>
export default {
    props: ['onClick', 'isShow'],

    data() {
        return {
            test: 123
        };
    },

    render() {
        return (
            <div class="test" onClick={ this.onClick }>
                { this.test }
                { this.isShow + '' }
            </div>
        );
    }
}
</script>

</script>
```
可以看到我们把jsx写在了render方法里面，render方法是vue2.0才支持的，用来提供对虚拟DOM的支持，也就是说只有vue2.0才支持jsx语法转换。

这里要注意的一点是vue里面编写jsx和在react里面的jsx语法还是有一点不一样的。
一下是一段覆盖大部分语法的vue jsx代码：
``` jsx
render (h) {
  return (
    <div
      // normal attributes or component props.
      id="foo"
      // DOM properties are prefixed with `domProps`
      domPropsInnerHTML="bar"
      // event listeners are prefixed with `on` or `nativeOn`
      onClick={this.clickHandler}
      nativeOnClick={this.nativeClickHandler}
      // other special top-level properties
      class={{ foo: true, bar: false }}
      style={{ color: 'red', fontSize: '14px' }}
      key="key"
      ref="ref"
      // assign the `ref` is used on elements/components with v-for
      refInFor
      slot="slot">
    </div>
  )
}
```
可以看到DOM属性要加domProps前缀，但这里lass和style却不需要，因为这两个是特殊的模块，而且react的class用的是className，vue却用的class。事件监听是以“on”或者“nativeOn”为开始。

实际上vue2.0的模版最后都会被编译为render方法，所以模版声明的组件和jsx声明的组件最后都是一样的。
上面的jsx最后会被编译成下面这样：
``` javascript
render (h) {
  return h('div', {
    // Component props
    props: {
      msg: 'hi'
    },
    // normal HTML attributes
    attrs: {
      id: 'foo'
    },
    // DOM props
    domProps: {
      innerHTML: 'bar'
    },
    // Event handlers are nested under "on", though
    // modifiers such as in v-on:keyup.enter are not
    // supported. You'll have to manually check the
    // keyCode in the handler instead.
    on: {
      click: this.clickHandler
    },
    // For components only. Allows you to listen to
    // native events, rather than events emitted from
    // the component using vm.$emit.
    nativeOn: {
      click: this.nativeClickHandler
    },
    // class is a special module, same API as `v-bind:class`
    class: {
      foo: true,
      bar: false
    },
    // style is also same as `v-bind:style`
    style: {
      color: 'red',
      fontSize: '14px'
    },
    // other special top-level properties
    key: 'key',
    ref: 'ref',
    // assign the `ref` is used on elements/components with v-for
    refInFor: true,
    slot: 'slot'
  })
}
```

这也意味着两种形式的组件是可以相互引用的。

有时候我们难免会在模版里引入jsx编写的vue组件或者在jsx编写的vue组件里引入模版组件，这里还是有些需要注意的事项：
1.在模版里面引入jsx的组件，可以通过components引用，另外props的编写从驼峰式改为连接符：
``` html
<template>
  <div class="wrapper">
    <Test :on-click="clickHandler" :is-show="show"></Test>
  </div>
</template>

<script>
import Test from './Test.vue';

export default {
  name: 'hello',
  components: {
    Test
  },
  data() {
    return {
      msg: 'Welcome to Your Vue.js App',
      show: true
    };
  },
  methods: {
    clickHandler(){
      this.show = !this.show;
    }
  }
};
</script>
```

2.在jsx里面引入vue模版组件，这里没有什么要注意的，除了连接符式的属性要转换成驼峰式，还有一个需要注意的是指令，如果用了jsx，那么内置的指令都不会生效（除了v-show），好在内置指令大部分都可以用jsx描述。那么自定义指令要怎么用呢？
自定义指令可以使用“v-name={value}”语法，如果要支持指令参数和modifier可以用“v-name={{ value, modifier: true }}”语法：
``` html
<script>
import Vue from 'vue';

Vue.directive('my-bold', {
  inserted: function (el) {
    el.style.fontWeight = 900;
  }
})

export default {
    props: ['onClick', 'isShow'],

    data() {
        return {
            test: 123
        };
    },

    methods: {
        afterLeave() {
            console.log('afterLeave')
        }
    },

    render() {
        const directives = [
            { name: 'my-bold', value: 666, modifiers: { abc: true } }
        ];

        return (
            <transition onAfterLeave={this.afterLeave} name="fade">
                <div class="test" onClick={this.onClick} v-show={ this.isShow } v-my-bold>
                    {this.test}
                    {this.isShow + ''}
                </div>
            </transition>
        );
    }
}
</script>
<style>
.fade-enter-active, .fade-leave-active {
  transition: opacity .5s
}
.fade-enter, .fade-leave-to {
  opacity: 0
}
</style>
```

### 扩展
如果有人觉得在vue组件里面要写data，props，computed和methods不够优雅，可以参考下这个插件[vue-class-component](https://github.com/vuejs/vue-class-component)，它能让你使用ES6的class和ES7的装饰器编写vue组件。

相关链接：
[babel-plugin-transform-vue-jsx](https://github.com/vuejs/babel-plugin-transform-vue-jsx)