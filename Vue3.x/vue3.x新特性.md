### v-for 中的 Ref 数组
在 Vue 2 中，在 v-for 里使用的 ref attribute 会用 ref 数组填充相应的 $refs property。当存在嵌套的 v-for 时，这种行为会变得不明确且效率低下。  
在 Vue 3 中，这样的用法将不再在 $ref 中自动创建数组。要从单个绑定获取多个 ref，请将 ref 绑定到一个更灵活的函数上 (这是一个新特性)
### 异步组件
+ 新的 defineAsyncComponent 助手方法，用于显式地定义异步组件
+ component 选项重命名为 loader
+ Loader 函数本身不再接收 resolve 和 reject 参数，且必须返回一个 Promise
```javascript
//2.x 异步组件是通过将组件定义为返回 Promise 的函数来创建
const asyncModal = () => import('./Modal.vue')
//带有选项的更高阶的组件语法
const asyncModal = {
  component: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  error: ErrorComponent,
  loading: LoadingComponent
}

//3.x 语法
// 不带选项的异步组件
const asyncModal = defineAsyncComponent(() => import('./Modal.vue'))
// 带选项的异步组件
const asyncModalWithOptions = defineAsyncComponent({
  loader: () => import('./Modal.vue'),
  delay: 200,
  timeout: 3000,
  errorComponent: ErrorComponent,
  loadingComponent: LoadingComponent
})

// 2.x 版本
const oldAsyncComponent = (resolve, reject) => {
  /* ... */
}
// 3.x 版本
const asyncComponent = defineAsyncComponent(
  () =>
    new Promise((resolve, reject) => {
      /* ... */
    })
)
```
### $children 移除
在 3.x 中，$children property 已移除，不再支持。如果你需要访问子组件实例，使用 $refs
### 自定义指令
指令的钩子函数已经被重命名，以更好地与组件的生命周期保持一致。 

__3.x语法__
+ created - 新的！在元素的 attribute 或事件侦听器应用之前调用。
+ bind → beforeMount
+ inserted → mounted
+ beforeUpdate：新的！这是在元素本身更新之前调用的，很像组件生命周期钩子。
+ update → 移除！有太多的相似之处要更新，所以这是多余的，请改用 updated。
+ componentUpdated → updated
+ beforeUnmount：新的！与组件生命周期钩子类似，它将在卸载元素之前调用。
+ unbind -> unmounted
```JavaScript
// 最终API如下
const MyDirective = {
  created(el, binding, vnode, prevVnode) {}, // 新增
  beforeMount() {},
  mounted() {},
  beforeUpdate() {}, // 新增
  updated() {},
  beforeUnmount() {}, // 新增
  unmounted() {}
}
```
```html
<p v-highlight="'yellow'">高亮显示此文本亮黄色</p>
```
```javascript
const app = Vue.createApp({})

app.directive('highlight', {
  beforeMount(el, binding, vnode) {
    el.style.background = binding.value
  }
})
```
### Data 选项
在 2.x 中，开发者可以定义 data 选项是 object 或者是 function。在 3.x，data 选项已标准化为只接受返回 object 的 function。  
__Mixin 合并行为变更__  
Mixin 提供了一种非常灵活的方式，来分发 Vue 组件中的可复用功能。一个 mixin 对象可以包含任意组件选项。当组件使用 mixin 对象时，所有 mixin 对象的选项将被“混合”进入该组件本身的选项。  
一个变动点：return的数据是一个对象时，只包含本组件对象的属性，return的是普通的属性则都包含。
### emits 选项
Vue 3 目前提供一个 emits 选项，和现有的 props 选项类似。这个选项可以用来定义组件可以向其父组件触发的事件。  
emits 可以是数组或对象，从组件触发自定义事件，emits 可以是简单的数组，也可以是对象，后者允许配置事件验证。  
在对象语法中，每个 property 的值可以为 null 或验证函数。验证函数将接收传递给 $emit 调用的其他参数。如果 this.$emit('foo',1) 被调用，foo 的相应验证函数将接收参数 1。__验证函数应返回布尔值，以表示事件参数是否有效__。点击 [这里](https://github.com/P-summer/Study-notes/blob/main/Vue3.x/emits%E5%92%8Csetup.md) 查看
### 事件 API
$on，$off 和 $once 实例方法已被移除，组件实例不再实现事件触发接口。  
+ 2.x 语法:在 2.x 中，Vue 实例可用于触发由事件触发 API 通过指令式方式添加的处理函数 ($on，$off 和 $once)。这可以创建 event bus，用来创建在整个应用程序中可用的全局事件监听器
+ 3.x 更新:我们从实例中完全移除了 $on、$off 和 $once 方法。$emit 仍然包含于现有的 API 中，因为它用于触发由父组件声明式添加的事件处理函数。
+ Event Bus：Event bus 模式可以被替换为实现了事件触发器接口的外部库，例如 mitt 或 tiny-emitter
