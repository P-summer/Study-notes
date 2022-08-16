## v-model
+ v-model本质上是语法糖，它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。
+ 处理组件时，判断 返回一个el.model对象，后续加在data字符串里，createComponent生成子组件时会对 data.model 的情况做处理（transformModel）：
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
+ processSlot先处理父组件，生成函数 接下来编译子组件，同样在 parser 阶段会执行 processSlot 处理函数
+ 先判断 template 上是否使用 scope 或 slot-scope，校验 slot-scope 是否和 v-for 同时使用
+ 处理 slot="xxx" 旧的具名插槽的写法，获取插槽名和动态插槽名
+ 处理2.6版本新用法，在 tempalte 标签上，得到 v-slot 的值，不同插槽语法禁止混合使用，得到插槽名（slotTarget），是否动态插槽和作用域插槽的值
+ 处理组件上的 v-slot，el 不是组件的话，提示 v-slot 只能出现在组件上或 template 标签上； 获取插槽名称以及是否为动态插槽，将每一个孩子的 parent 属性都设置为 slotContainer（也就是 template 元素）
+ 不要在 slot 标签上使用 key 属性

#### generate生成code
+ genData 会给 data 添加一个 slot 属性，并指向 slotTarget; 
+ genSlot生产 ```_t``` 函数， ```_t``` 函数对应的就是 renderSlot，获取插槽内容，有就返回，没有就返回默认内容（this.$slots(vm.$slots)在initRender通过resolveSlots获取）

```javascript
if (el.slotTarget && !el.slotScope) {
  data += `slot:${el.slotTarget},`
}
```
### keep-alive
+ <keep-alive> 是 Vue内置组件，它有一个属性 abstract 为 true，直接实现了 render 函数，执行 <keep-alive> 组件渲染的时候，就会执行到这个 render 函数
+ 由于我们也是在 <keep-alive> 标签内部写 DOM，所以可以先获取到它的默认插槽，然后再获取到它的第一个子节点。进行匹配，不匹配返回VNode，否则进行缓存
+ 命中缓存，拿缓存，调整顺序放最后一个，否则把 vnode 设置进缓存，如果配置max且超过则从 cache 删除第一个
+ 组件渲染，patch --> createComponent 首次渲染和普通组件一样
+ 当数据发送变化,patchVnode 在做各种 diff 之前，会先执行 prepatch更新组件触发keepalive的render  patch
+ 并且在执行 init 钩子函数的时候不会再执行组件的 mount 过程了
```javascript
 // 第一次渲染的时候，vnode.componentInstance 为 undefined，vnode.data.keepAlive 为 true，因为它的父组件 <keep-alive> 的 render 函数会先执行，
 // 那么该 vnode 缓存到内存中，并且设置 vnode.data.keepAlive 为 true，因此 isReactivated 为 false，那么走正常的 init 的钩子函数执行组件的 mount
 const isReactivated = isDef(vnode.componentInstance) && i.keepAlive
```








