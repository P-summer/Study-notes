### 工厂模式
工厂模式定义一个用于创建对象的接口，这个接口由子类决定实例化哪一个类。每次返回的都是一个全新的实例
```JavaScript
//原理部分
class Axios {}
class A {
  create() {
    return new Axios()
  }
}
const axios = new A()
export default axios

//使用部分
import axios from 'axios'

// 创建很多实例
const httpRequest1 = axios.create()
const httpRequest2 = axios.create()
const httpRequest3 = axios.create()
```

### 单例模式
一个类只有一个实例，并提供一个访问它的全局访问点。并且整个项目仅此这一个实例
```JavaScript
// 定义一个类  utils/request.ts
class HttpRequest {
  instance: AxiosInstance;
  constructor(options: CreateAxiosOptions) {
    this.instance = axios.create(options)
  }

  setHeader() {...}
  get() {...}
  post() {...}
  put() {...}
  delete() {...}
}
// 生成一个实例
const request = new HttpRequest({})

// 全局仅用这么一个请求实例
export default request


import request '@/utils/request'

const fetchData = (url) => {
  return request.get(url)
}
```

### 策略模式
根据不同的策略去做不同的事情
```JavaScript
const doMap: Record<number, Function> = {
  20: () => { // do something },
  30: () => { // do something },
  40: () => { // do something },
}

const doSomething = (age: number) => {
  doMap[age]?.()
}
```

### 适配器模式
+ 将一个类的接口转化为另外一个接口，以满足用户需求，使类之间接口不兼容问题通过适配器得以解决。
+ 可以让任何两个没有关联的类一起运行。提高了类的复用(computed)
```JavaScript
class Plug {
  getName() {
    return 'iphone充电头';
  }
}

class Target {
  constructor() {
    this.plug = new Plug();
  }
  getName() {
    return this.plug.getName() + ' 适配器Type-c充电头';
  }
}

let target = new Target();
target.getName(); // iphone充电头 适配器转Type-c充电头
```
### 装饰器模式
定义一个类，在不改这个类的前提下，给这个类拓展功能
```JavaScript
class Cellphone {
    create() {
        console.log('生成一个手机')
    }
}
class Decorator {
    constructor(cellphone) {
        this.cellphone = cellphone
    }
    create() {
        this.cellphone.create()
        this.createShell(cellphone)
    }
    createShell() {
        console.log('生成手机壳')
    }
}
// 测试代码
let cellphone = new Cellphone()
cellphone.create()

console.log('------------')
let dec = new Decorator(cellphone)
dec.create()
```

### 代理模式
为对象提供一种代理，便以控制对这个对象的访问，不能直接访问目标对象：ES6 Proxy
```JavaScript
const handler = {
    get: function(obj, prop) {
        return prop in obj ? obj[prop] : 7;
    }
};

const p = new Proxy({}, handler);
p.a = 1;
p.b = undefined;

console.log(p.a, p.b);      // 1, undefined
console.log('c' in p, p.c); // false, 37
```

### 观察者模式
+ 定义对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都将得到通知
+ Vue 响应式原理

### 发布订阅模式
发布订阅模式跟观察者模式很像，他们其实都有发布者和订阅者，但是他们是有区别的
+ 观察者模式的发布和订阅是互相依赖的
+ 发布订阅模式的发布和订阅是不互相依赖的，因为有一个统一调度中心（Vue EentBus）
