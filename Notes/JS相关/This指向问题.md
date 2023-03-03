### This的5中绑定
+ 默认绑定（非严格模式下指向全局对象，严格模式下this会绑定到underfind）
+ 隐式绑定（当函数引用有 __上下文对象__ 时，如obj.foo()的调用方式，foo内的this指向obj）
+ 显示绑定（通过call()或者apply()方法直接指定this的绑定对象，如fool.call(obj)）
+ new绑定
+ 箭头函数绑定(this的指向有外层作用域决定)  
 
