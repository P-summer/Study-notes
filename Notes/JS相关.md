### JS原型链
引用类型：Object、Array、Function、Date、RegExp  
+ 引用类型，都具有对象特性，即可自由扩展属性。
```javascript
const obj = {}
const arr = []
const fn = function () {}

obj.a = 1
arr.a = 1
fn.a = 1

console.log(obj.a) // 1
console.log(arr.a) // 1
console.log(fn.a) // 1
```
+ 引用类型，都有一个隐式原型__proto__属性，属性值是一个普通的对象。
```javascript
const obj = {};
const arr = [];
const fn = function() {}

console.log('obj.__proto__', obj.__proto__);
console.log('arr.__proto__', arr.__proto__);
console.log('fn.__proto__', fn.__proto__);
```
+ 引用类型，隐式原型__proto__的属性值指向它的构造函数的显式原型 prototype 属性值。
```javascript
const obj = {};
const arr = [];
const fn = function() {}

obj.__proto__ == Object.prototype // true
arr.__proto__ === Array.prototype // true
fn.__proto__ == Function.prototype // true
```
+ 当你试图得到一个对象的某个属性时，如果这个对象本身没有这个属性，那么它会去它的隐式原型__proto__（也就是它的构造函数的显式原型 prototype）中寻找。  
```javascript
const obj = { a:1 }
obj.toString
// ƒ toString() { [native code] }
// obj 对象并没有 toString 属性，之所以能获取到 toString 属性，是遵循了第四条规则，从它的构造函数 Object 的 prototype 里去获取。
```
### instanceof 原理
instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。typeof返回string。  
```javascript
function Car(make, model, year) {
  this.make = make;
  this.model = model;
  this.year = year;
}
const auto = new Car('Honda', 'Accord', 1998);

console.log(auto instanceof Car);
// expected output: true
console.log(auto instanceof Object);
// expected output: true

// 手写instanceof
function instanceof(L,R) {
  // 左边取__proto__,右边取原型
  let leftObj = L.__proto__,rightObj = R.prototype;
  while(true) {
    if (leftObj === null) {
      return false;
    } else if (leftObj === rightObj) {
      return true;
    }
    leftObj = leftObj.__proto__;//继续向上一层查找
  }
}
```
### JS继承
