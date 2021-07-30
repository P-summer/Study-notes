### 一个新的全局 API：createApp
+ 调用 createApp 返回一个应用实例，这是 Vue 3 中的新概念
```javascript
import { createApp } from 'vue'
const app = createApp({})
```
+ config.productionTip 移除(警告提示)
+ Vue.prototype 替换为 config.globalProperties
在 Vue 2 中， Vue.prototype 通常用于添加所有组件都能访问的 property。  
在 Vue 3 等同于config.globalProperties。这些 property 将被复制到应用中作为实例化组件的一部分。  
+ 插件使用者须知，Vue.use 全局 API 在 Vue 3 中不再使用
```javascript
// 2.x
Vue.use(VueRouter)
// 3.x
const app = createApp(MyApp)
app.use(VueRouter)
```
### 全局 API Treeshaking
+ 在 Vue 3 中，全局和内部 API 都经过了重构，并考虑到了 tree-shaking 的支持。因此，全局 API 现在只能作为 ES 模块构建的命名导出进行访问  
+ 直接调用 Vue.nextTick()将会报 undefined is not a function 的错误
```javascript
import { nextTick } from 'vue'

nextTick(() => {
  // 一些和DOM有关的东西
})
```
__受影响的 API:__  
+ Vue.nextTick
+ Vue.observable (用 Vue.reactive 替换)
+ Vue.version
+ Vue.compile (仅完整构建版本)
+ Vue.set (仅兼容构建版本)
+ Vue.delete (仅兼容构建版本)
### key attribute
+ 新增：对于 v-if/v-else/v-else-if 的各分支项 key 将不再是必须的，因为现在 Vue 会自动生成唯一的 key。  
非兼容：如果你手动提供 key，那么每个分支必须使用唯一的 key。你不能通过故意使用相同的 key 来强制重用分支。
+ 非兼容：```<template v-for>``` 的 key 应该设置在 ```<template>``` 标签上 (而不是设置在它的子节点上)
```html
<!-- 在 Vue 2.x 中 <template> 标签不能拥有 key。不过你可以为其每个子节点分别设置 key -->
<template v-for="item in list">
  <div :key="'heading-' + item.id">...</div>
  <span :key="'content-' + item.id">...</span>
</template>
<!-- 在 Vue 3.x 中 key 则应该被设置在 <template> 标签上-->
<template v-for="item in list" :key="item.id">
  <div>...</div>
  <span>...</span>
</template>
```
### 按键修饰符
+ 非兼容：不再支持使用数字 (即键码) 作为 v-on 修饰符
+ 非兼容：不再支持 config.keyCodes
### 移除 $listeners
### 挂载API变化
应用挂载到拥有匹配 id="app" 的 div 的页面时
```html
<!--2.x-->
<body>
  <div id="rendered">Hello Vue!</div>
</body>
<!--3.x-->
<body>
  <div id="app" data-v-app="">
    <div id="rendered">Hello Vue!</div>
  </div>
</body>
```
### propsData移除
propsData 选项之前用于在创建 Vue 实例的过程中传入 prop，现在它被移除了。  
propsData 选项已经被移除。如果需要在实例创建时向根组件传入 prop，你应该使用 createApp 的第二个参数：
```javascript
const app = createApp(
  {
    props: ['username'],
    template: '<div>{{ username }}</div>'
  },
  { username: 'Evan' }
)
```
### 渲染函数API
+ 渲染函数参数
```js
// 在 2.x 中，render 函数会自动接收 h 函数 (它是 createElement 的惯用别名) 作为参数
export default {
  render(h) {
    return h('div')
  }
}
// 在 3.x 中，h 函数现在是全局导入的，而不是作为参数自动传递
import { h } from 'vue'
export default {
  render() {
    return h('div')
  }
}
```
+ 在 3.x 中，由于 render 函数不再接收任何参数，它将主要在 setup() 函数内部使用。这还有一个好处：可以访问在作用域中声明的响应式状态和函数，以及传递给 setup() 的参数
```js
import { h, reactive } from 'vue'
export default {
  setup(props, { slots, attrs, emit }) {
    const state = reactive({
      count: 0
    })
    function increment() {
      state.count++
    }
    // 返回渲染函数
    return () =>
      h(
        'div',
        {
          onClick: increment
        },
        state.count
      )
  }
}
```
+ 注册组件
```js
// 在 2.x 中，注册一个组件后，把组件名作为字符串传给渲染函数的第一个参数，渲染函数可以正常的工作
Vue.component('button-counter', {
  data() {
    return {
      count: 0
    }
  }
  template: `
    <button @click="count++">
      Clicked {{ count }} times.
    </button>
  `
})

export default {
  render(h) {
    return h('button-counter')
  }
}
// 在 3.x 中，由于 VNode 是上下文无关的，不能再用字符串 ID 隐式查找已注册组件。相反地，需要使用一个导入的 resolveComponent 方法
import { h, resolveComponent } from 'vue'

export default {
  setup() {
    const ButtonCounter = resolveComponent('button-counter')
    return () => h(ButtonCounter)
  }
}
```
