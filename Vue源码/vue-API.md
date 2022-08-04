## v-model
+ v-model本质上是语法糖，它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。
+ 处理组件时，判断 返回一个el.model对象，后续加在data字符串里，生成子组件时会对 data.model 的情况做处理（transformModel）：
给 data.props 添加 data.model.value，并且给data.on 添加 data.model.callback

```javascript
<input type='radio' value='1' v-model='foo'/>
// 等价于
<input type='radio' value='1'
:checked="foo == '1'"
@change="foo = $event.target.value" />

// v-model数据双向绑定原理：给 el 添加一个 prop，相当于我们在 input 上动态绑定了 value，又给 el 添加了事件处理，相当于在 input 上绑定了 input 事件
addProp(el, 'value', `(${value})`)
addHandler(el, event, code, null, true)

<input
  v-bind:value="message"
  v-on:input="message=$event.target.value"
>
// v-model组件  生成code
el.model = {
  callback:'function ($$v) {message=$$v}',
  expression:'"message"',
  value:'(message)'
}
// 创建组件
data.props = {
  value: (message),
}
data.on = {
  input: function ($$v) {
    message=$$v
  }
} 
```
vue2给组件提供了 __model__ 属性，可以让用户自定义 __传值的prop名__ 和 __更新值的事件名__ 。  
v-model是 __双向绑定，单项数据流__ (子组件不能改变父组件传递给它的prop属性，推荐做法是抛出事件，通知父组件自行改变)  
+ 让开发的组件支持v-model,在定义vue组件时，可以提供一个model属性，来定义该组件以何种方式支持v-model。
```javascript
// model属性的默认值 
export default {
  model: {
    prop: 'value',
    event: 'input'
  }
}
```
## slot
```html
<div class="container">
  <header>
    <slot name="header" v-bind:user="info"></slot>
  </header>
  <main>
    <slot></slot>
  </main>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>

// 2.6
<base-layout>
  <template v-slot:header v-slot:header="props">
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>

// 2.5及以前
<template slot="header" slot-scope="slotProps">
  <h1>Here might be a page title</h1>
</template>
// 或者可用放普通元素
<h1 slot="header">Here might be a page title</h1>

```
#### 编译解析
+ 先判断 template 上是否使用 scope 或 slot-scope，校验 slot-scope 是否和 v-for 同时使用
+ 处理 slot="xxx" 旧的具名插槽的写法，获取插槽名和动态插槽名
+ 处理2.6版本新用法，在 tempalte 标签上，得到 v-slot 的值，不同插槽语法禁止混合使用，得到插槽名，是否动态插槽和作用域插槽的值
+ 处理组件上的 v-slot，el 不是组件的话，提示 v-slot 只能出现在组件上或 template 标签上； 获取插槽名称以及是否为动态插槽，将每一个孩子的 parent 属性都设置为 slotContainer（也就是 template 元素）
+ 不要在 slot 标签上使用 key 属性

