#### 1.从浏览器输入URL到整个页面显示经历了什么？
待补充
#### 2.EventLoop？
待补充
#### 3.跨域问题及解决方式？
eg
#### 4.call、apply、bind 三者的区别？call 和 apply 哪个性能会好一些？如何实现 call、apply、bind？
bind、apply、call 都是用来绑定函数执行时this的指向（改变函数执行时的上下文），同时还可以传参，调用它们的对象必须是一个函数 Function。call、apply可用于调用函数
```javascript
// call  
Function.call(obj, arg1, arg2, arg3, ...);

// apply，有两个参数，第二个是类数组
Function.apply(obj, [argArray]);

// bind，bind 方法的返回值是函数，需要手动调用才会执行，而 apply 和 call 则是立即调用
Function.bind(obj, arg1, arg2, arg3, ...);
```
```javascript
const obj = {
  name: '小鸭子',
};

function say (arg1, arg2) {
  console.log(`${this.name ? `我是一只${this.name}，` : ''}${arg1}，${arg2}`);
}

say('咿呀咿呀哟', '呱呱！'); // 咿呀咿呀哟，呱呱！
say.call(obj, '咿呀咿呀哟', '呱呱！') // 我是一只小鸭子，咿呀咿呀哟，呱呱！
say.apply(obj, ['咿呀咿呀哟', '呱呱！']); // 我是一只小鸭子，咿呀咿呀哟，呱呱！

const manualSay = say.bind(obj, '咿呀咿呀哟', '呱呱！'); // 绑定但不是立即执行
manualSay(); // 手动执行，输出：我是一只小鸭子，咿呀咿呀哟，呱呱！
```
**性能比较：** call 的性能要比 apply 好一些（尤其是传递给函数的参数超过三个的时候）。  [其他实现](https://leetoffer.com/question/608fbbb4a8cba06305b045f8)
```javascript
// call 实现
Function.prototype.myCall = function(context) {
  context = context || window; // 默认 window
  const args = [...arguments].slice(1); // 参数
  const fn = Symbol(); // 给 context 设置一个独一无二的属性，避免覆盖原有属性
  context[fn] = this; // 这里的 this 指向调用它的函数 say
  // 调用  此时say方法里面的this指向context，如果say方法是箭头函数的话，就会指向window，亲测
  const result = context[fn](...args);
  delete context[fn]; // 删除添加的属性
  return result; // 返回值
}

say.myCall(obj, '咿呀咿呀哟', '呱呱！') // 我是一只小鸭子，咿呀咿呀哟，呱呱！
```
#### 5.浏览器内存泄露的场景及如何解决？
**什么是内存泄漏** ：申请的内存没有及时回收掉，被泄漏了  
**为什么会发生内存泄漏** ：虽然前端有垃圾回收机制，但当某块无用的内存，却无法被垃圾回收机制认为是垃圾时，也就发生内存泄漏了.垃圾回收机制通常是使用标志清除策略  

__意外的全局变量__ 全局变量的生命周期最长，直到页面关闭前，它都存活着，所以全局变量上的内存一直都不会被回收.当全局变量使用不当，没有及时回收（手动赋值 null），或者拼写错误等将某个变量挂载到全局变量时，也就发生内存泄漏了。  
__定时器__ setTimeout 和 setInterval 是由浏览器专门线程来维护它的生命周期，所以当在某个页面使用了定时器，页面销毁时，没有手动去释放清理这些定时器的话，那么这些定时器还是存活着的。  
__使用不当的闭包__ 闭包可以维持函数内局部变量，使其得不到释放。  
__回调函数__ 定时器，事件监听器，Ajax请求，跨窗口通讯，Web Workers或者其他同步、异步任务，只要使用回调函数，实际上就是闭包。  
__循环引用__ eg:两个对象，互相引用对方。如果持续调用下面的函数，会导致内存不断升高  
__DOM元素的引用__：DOM 元素的生命周期正常是取决于是否挂载在 DOM 树上，当从 DOM 树上移除时，也就可以被销毁回收了。但如果某个 DOM 元素，在 js 中也持有它的引用时，那么它的生命周期就由 js 和是否在 DOM 树上两者决定了，记得移除时，两个地方都需要去清理才能正常回收它。  
