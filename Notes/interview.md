##### 1.从浏览器输入URL到整个页面显示经历了什么？
待补充
##### 2.EventLoop？
待补充
##### 3.跨域问题及解决方式？
eg
##### 4.call、apply、bind 三者的区别？call 和 apply 哪个性能会好一些？如何实现 call、apply、bind？
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
__性能比较__：call 的性能要比 apply 好一些（尤其是传递给函数的参数超过三个的时候）。  [实现](https://leetoffer.com/question/608fbbb4a8cba06305b045f8)
