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
