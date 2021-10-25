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
### JS new操作符
+ 创建一个空对象  
+ 让空对象的 __proto__ (IE 没有该属性) 成员指向构造函数的 prototype 成员对象  
+ 使用 apply 调用构造函数，属性和方法被添加到 this 引用的对象中  
+ 如果构造函数中没有返回其他对象，那么返回 this，即创建的这个新对象；否则，返回构造函数返回的对象  
```javascript
function create(Con.prototype, ...args) {
  let obj = {};
  obj.__proto__ == Con.prototype;
  let result = Con.apply(obj, args);
  return result instanceof Object ? result : obj;
}
```
### JS继承
```javascript
function Father(name) {
  // 实例属性
  this.name = name
  // 实例方法
  this.sayName = function() {
    console.log(this.name)
  }
}
// 原型属性
Father.prototype.age = 19
// 原型方法
Father.prototype.sayAge = function() {
  console.log(this.age)
}
```
+ 原型链继承：将父类的实例作为子类的原型
```javascript
function Son(name) {
  this.name = name
}
Son.prototype = new Father()
const son = new Son('son')
son.sayName() // son
son.sayAge() // 19
//  简单，易于实现;父类新增原型方法、原型属性，子类都能访问到
//  无法实现多继承，因为原型一次只能被一个实例更改;来自原型对象的所有属性被所有实例共享;创建子类实例时，无法向父构造函数传参
```
+ 构造继承:复制父类的实例属性给子类
```javascript
function Son(name) {
  Father.call(this, "Son props")
  this.name = name
}
const son = new Son('son')
son.sayName() // son
son.sayAge() // 报错，无法继承父类原型
console.log(son instanceof Son) // true
console.log(son instanceof Father) // false
//  创建子类实例时，可以向父类传参;可以实现多继承（call 多个父类对象）
//  实例并不是父类的实例，只是子类的实例;只能继承父类实例属性和方法，不能继承原型属性和方法
```
+ 组合继承:将原型链和构造函数组合一起，使用原型链实现对原型属性和方法的继承，使用构造函数实现实例属性继承
+ 实例继承:为父类实例添加新特性，作为子类实例返回
```javascript
function Son(name) {
  const father = new Father('Son Props')
  father.name = name
  return father
}

const son = new Son('son')
son.name // son
son.sayAge() // 19
son.sayName // son
son instanceof Father // true
son instanceof Son // false
son.constructor === Father // true
son.constructor === Son // false
// 不限制调用方式;  实例是父类的实例，不是子类的实例;不支持多继承
```
+ 拷贝继承：对父类实例中的方法和属性拷贝给子类的原型
+ 寄生组合继承：通过寄生方式，砍掉父类的实例属性，避免组合继承生成两份实例的缺点
```javascript
function Son(name) {
  Father.call(this)
  this.name = name
}
// Object.create()方法创建一个新对象，使用现有的对象来提供新创建的对象的__proto__。
Son.prototype = Object.create(Father.prototype)
Son.prototype.constructor = Son
const son = new Son('son')
son.sayAge // 19
son.sayName // son
son instanceof Father // true
son instanceof Son // true
son.constructor === Father // false
son.constructor === Son // true
// 比较完美（js 实现继承首选方式）;实现方式较为复杂
```
### var、const、let 对比
+ 在同一个作用域内，var 可以重复声明变量，let、const 不能重复声明同一个变量。ES5 是函数作用域，即一个函数内就是一个作用域，ES6 是块级作用域，花括号内就是一个作用域。
+ var 有变量提升，可以在变量声明前使用，let、const 不存在变量提升，在变量前使用会报错
+ let、const 有暂时性死区，即父作用域中有 var 定义，在块作用域中又定义了 let、const，那再 let、const 之前使用就是死区
+ const 必须在声明的时候赋值；const 声明的变量不能再修改







