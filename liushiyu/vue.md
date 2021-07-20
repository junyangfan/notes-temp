---
title: liushiyu
tags: 新建,模板,小书匠
renderNumberedHeading: true
grammar_cjkRuby: true
---


### new Vue(options) 发生了什么？

1）处理组件的配置项

初始化根组件时进行了选项合并操作，即将全局的配置项合并到根组件的局部配置上 初始化子组件的时候做了一些性能优化，将组件配置对象上的一些深层次的属性放到vm.$options 选项中，即拍平配置项 减少运行时原型链的查找，提高代码的执行效率。

2）初始化组件实例的关系属性（initLifeCycle）比如：$parent、$root、$children、$refs等

3）处理自定义事件 注：<comp @click="handleClick" /> 上注册的事件，实际上是子组件在监听，即遵循谁触发谁监听的原则，与父组件无关。

4）初始化插槽 获取this.$slots 定义 this._c 即createElement 方法 平时使用的h函数

5）调用 beforeCreate 钩子函数

6）初始化组件的inject配置项，得到result[key] = val 形式的配置对象，然后对该配置对象做响应式处理，并将每个key代理到vm实例上

7）响应式数据初始化，处理props、methods、data、computed、watch等选项

8）解析组件配置项上的provide对象，将其挂载到vm._provided属性上

9）调用created 钩子函数

10）如果发现配置项上有el选项，则自动调用$mount方法，反之需要手动调用$mount方法

接下来则进入挂载阶段


### beforeCreate中可以拿到数据吗？

不能， beforeCreate 生命周期执行的时候，数据还没有开始初始化，在它之前只做了一些关于属性的初始化和关于自定义事件的初始化还有插槽的初始化和_c 函数的初始化即h函数

### 最早拿到数据的生命周期是哪个？

created

因为在 beforeCreate生命周期之后 created  之前 完成了 state数据的初始化 和 provide提供的数据的初始化