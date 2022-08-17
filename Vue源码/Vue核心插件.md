### Vue-Router
+ Vue 提供了 Vue.use 的全局 API 来注册这些插件,Vue-Router其中定义了 VueRouter 类,也实现了 install 的静态方法
+ 也实现了 install 的静态方法,  install 方法里面利用 Vue.mixin 去把 beforeCreate 和 destroyed 钩子函数注入到每一个组件中。
+ beforeCreate 执行了init方法，所以组件在执行 beforeCreate 钩子函数的时候，如果传入了 router 实例，都会执行 router.init 方法
+ init 存储了vue实例，判断 路由模式（有hash，history，abstract三种），会执行 history.transitionTo 方法做路由过渡，进而引出了 matcher 的概念

```javascript
export default class VueRouter {
  static install: () => void
  constructor (options: RouterOptions = {}) {}
}
// install 精简代码
Vue.mixin({
  beforeCreate () {
    this._router.init(this)
  },
  destroyed () {
    registerInstance(this)
  }
})
```

### vuex
+  Vuex 的初始化过程，它包括安装、Store 实例化过程 2 个方面,Vuex 也同样存在一个静态的 install 方法
+  我们在 import Vuex 之后，会实例化其中的 Store 对象，返回 store 实例并传入 new Vue 的 options 中;Store 对象的构造函数接收一个对象参数，它包含 actions、getters、state、mutations、modules
+  Vuex 定义了 class Store 实例化过程拆成 3 个部分，分别是初始化模块，安装模块和初始化 ```store._vm```
+  构造好这个 store 后，需要提供一些 API 对这个 store 做存取的操作,Vuex 最终存储的数据是在 state 上的

```jvavascript
class Store {
  constructor (options = {}) {
    this._modules = new ModuleCollection(options);
    installModule(this, state, [], this._modules.root);
    resetStoreVM(this, state);
  }
}
```
