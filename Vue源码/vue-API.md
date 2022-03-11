### v-model
v-model本质上是语法糖，它负责监听用户的输入事件以更新数据，并对一些极端场景进行一些特殊处理。  
```javascript
<input type='radio' value='1' v-model='foo'/>
// 等价于
<input type='type' value='1'
:checked="foo == '1'"
@change="foo = $event.target.value" />
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
