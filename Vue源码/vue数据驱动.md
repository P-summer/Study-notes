### Runtime Only VS Runtime + Compiler  
通常我们利用 vue-cli 去初始化我们的 Vue.js 项目的时候会询问我们用 Runtime Only 版本的还是 Runtime + Compiler 版本。  
+ __Runtime Only__  
我们在使用 Runtime Only 版本的 Vue.js 的时候，通常需要借助如 webpack 的 vue-loader 工具把 .vue 文件编译成 JavaScript，因为是在编译阶段做的，
所以它只包含运行时的 Vue.js 代码，因此代码体积也会更轻量。  
+ __Runtime + Compiler__  
我们如果没有对代码做预编译，但又使用了 Vue 的 template 属性并传入一个字符串，则需要在客户端编译模板。  
```javascript
// 需要编译器的版本
new Vue({
  template: '<div>{{ hi }}</div>'
})
// 这种情况不需要
new Vue({
  render (h) {
    return h('div', this.hi)
  }
})
```
因为在 Vue.js 2.0 中，最终渲染都是通过 render 函数，如果写 template 属性，则需要编译成 render 函数，那么这个编译过程会发生运行时，所以需要带有编译器的版本。  
很显然，这个编译过程对性能会有一定损耗，所以通常我们更推荐使用 Runtime-Only 的 Vue.js。

### vue实例挂载的实现
Vue 中我们是通过 $mount 实例方法去挂载 vm 的，$mount 方法在多个文件中都有定义，如 src/platform/web/entry-runtime-with-compiler.js、
src/platform/web/runtime/index.js、src/platform/weex/runtime/index.js。因为 $mount 这个方法的实现是和平台、构建方式都相关的。  

__compiler 版本的 $mount 实现非常有意思，先来看一下 src/platform/web/entry-runtime-with-compiler.js 文件中定义：__  
这段代码首先缓存了原型上的 $mount 方法，再重新定义该方法，我们先来分析这段代码。首先，它对 el 做了限制，Vue 不能挂载在 body、
html 这样的根节点上。接下来的是很关键的逻辑 —— __如果没有定义 render 方法，则会把 el 或者 template 字符串转换成 render 方法__。
这里我们要牢记，在 Vue 2.0 版本中，所有 Vue 的组件的渲染最终都需要 render 方法，无论我们是用单文件 .vue 方式开发组件，
还是写了 el 或者 template 属性，最终都会转换成 render 方法，那么这个过程是 Vue 的一个“在线编译”的过程，它是调用
__compileToFunctions__ 方法实现的，编译过程我们之后会介绍。最后，调用原先原型上的 $mount 方法挂载。  

__原先原型上的 $mount 方法在 src/platform/web/runtime/index.js 中定义，之所以这么设计完全是为了复用，因为它是可以被 runtime only 版本的 Vue 直接使用的。__
```javascript
// public mount method
Vue.prototype.$mount = function (
  el?: string | Element,
  hydrating?: boolean
): Component {
  el = el && inBrowser ? query(el) : undefined
  return mountComponent(this, el, hydrating)
}
```
$mount 方法支持传入 2 个参数，第一个是 el，它表示挂载的元素，可以是字符串，也可以是 DOM 对象，如果是字符串在浏览器环境下会调用 query 方法转换成 DOM 对象的。
第二个参数是和服务端渲染相关，在浏览器环境下我们不需要传第二个参数。  
$mount 方法实际上会去调用 ```mountComponent``` 方法，这个方法定义在 src/core/instance/lifecycle.js 文件中  
```mountComponent``` 核心就是先实例化一个渲染Watcher，在它的回调函数中会调用 updateComponent 方法，在此方法中调用 ```vm._render``` 方法先生成虚拟 Node，
最终调用 ```vm._update``` 更新 DOM。  
函数最后判断为根节点的时候设置 vm._isMounted 为 true， 表示这个实例已经挂载了，同时执行 mounted 钩子函数。 这里注意 vm.$vnode 表示 Vue 实例的父虚拟 Node，
所以它为 Null 则表示当前是根 Vue 的实例。

