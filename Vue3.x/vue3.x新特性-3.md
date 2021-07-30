### 插槽统一
此更改统一了 3.x 中的普通插槽和作用域插槽。
+ this.$slots 现在将插槽作为函数公开
+ 非兼容：移除 this.$scopedSlots
### 过渡的 class 名更改
过渡类名 v-enter 修改为 v-enter-from、过渡类名 v-leave 修改为 v-leave-from。  
```<transition>``` 组件相关属性名也发生了变化：
+ leave-class 已经被重命名为 leave-from-class (在渲染函数或 JSX 中可以写为：leaveFromClass)
+ enter-class 已经被重命名为 enter-from-class (在渲染函数或 JSX 中可以写为：enterFromClass)
### Transition 作为 Root
使用一个 ```<transition>``` 作为一个组件的根节点，当组件从外部被切换时将不再触发过渡效果。
### Transition Group 根元素
```<transition-group>``` 不再默认渲染根元素，但仍然可以用 tag prop 创建根元素。  
__2.x 语法__
在 Vue 2 中，```<transition-group>``` 像其它自定义组件一样，需要一个根元素。默认的根元素是一个 ```<span>```，但可以通过 tag prop 定制。
```html
<transition-group tag="ul">
  <li v-for="item in items" :key="item">
    {{ item }}
  </li>
</transition-group>
```
__3.x 语法__
在 Vue 3 中，我们有了片段的支持，因此组件不再需要根节点。所以，```<transition-group>``` 默认不再渲染根节点。
+ 如果像上面的示例一样，已经在 Vue 2 代码中定义了 tag prop，那么一切都会和之前一样
+ 如果没有定义 tag prop，而且样式或其它行为依赖于 ```<span>``` 根元素的存在才能正常工作，那么只需将 tag="span" 添加到 ```<transition-group>```：
```
<transition-group tag="span">
  <!-- -->
</transition-group>
```
### 移除 v-on.native 修饰符
v-on 的 .native 修饰符已被移除。
+ __2.x 语法__ 默认情况下，传递给带有 v-on 的组件的事件监听器只有通过 this.$emit 才能触发。要将原生 DOM 监听器添加到子组件的 __根元素__ 中，可以使用 .native 修饰符
```html
<my-component
  v-on:close="handleComponentEvent"
  v-on:click.native="handleNativeClickEvent"
/>
```
+ 新增的 emits 选项允许子组件定义真正会被触发的事件。







