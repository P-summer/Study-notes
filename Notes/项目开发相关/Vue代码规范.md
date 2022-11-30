+ 组件名为多个单词
```javascript
Vue.component('todo-item', {
  // ...
})
export default {
  name: 'TodoItem',
  // ...
}
components/
|- MyComponent.vue
components/
|- my-component.vue
```
+ 组件的 data 必须是一个函数。
+ Prop 定义应该尽量详细。
```javascript
props: {
  status: String
}
// 更好的做法！
props: {
  status: {
    type: String,
    required: true,
    validator: function (value) {
      return [
        'syncing',
        'synced',
        'version-conflict',
        'error'
      ].indexOf(value) !== -1
    }
  }
}
```
+ 为 v-for 设置键值,总是用 key 配合 v-for。
+ 永远不要把 v-if 和 v-for 同时用在同一个元素上。
+ 为组件样式设置作用域
+ 基础组件名:应用特定样式和约定的基础组件 (也就是展示类的、无逻辑的或无状态的组件) 应该全部以一个特定的前缀开头，比如 Base、App 或 V
+ 单例组件名:只应该拥有单个活跃实例的组件应该以 The 前缀命名，以示其唯一性(每个页面只使用一次，不接收prop)。
+ 
