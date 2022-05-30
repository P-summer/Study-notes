```JavaScript
//core/instance/index	vue初始化方法
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword')
  }
  this._init(options)
}

initMixin(Vue)

// core/instance/init.js
let uid = 0;
export function initMixin(Vue: Class<Component>) {
  Vue.prototype._init = function(options?: Object) {
    const vm: Component = this;

    vm._uid = uid++; // 当前实例的 _uid 加 1

	//  a flag to avoid this being observed
    // 用 _isVue 来标识当前实例是 Vue 实例， 这样做是为了后续被 observed
    vm._isVue = true;
    
    // merge options 合并options 
    if (options && options._isComponent) { // _isComponent 标识当前为 内部Component
      // 内部Component 的 options 初始化
      initInternalComponent(vm, options); 
    }
    else { // 非内部Component的 options 初始化
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      );
    }
    
    // 在render中将this指向vm._renderProxy
    if (process.env.NODE_ENV !== 'production') {
      initProxy(vm);
    }
    else {
      vm._renderProxy = vm;
    }
    // expose real self
    vm._self = vm;
    initLifecycle(vm); // 初始化生命周期
    initEvents(vm); // 初始化事件
    initRender(vm); // 初始化渲染函数
    callHook(vm, 'beforeCreate'); // 回调 beforeCreate 钩子函数
    initInjections(vm); // resolve injections before data/props
    // 初始化 vm 的状态
    initState(vm);
    initProvide(vm); // resolve provide after data/props
    callHook(vm, 'created'); // vm 已经创建好 回调 created 钩子函数

    if (vm.$options.el) { // 挂载实例
      vm.$mount(vm.$options.el);
    }
  };
}
```
