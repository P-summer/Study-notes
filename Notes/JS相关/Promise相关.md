### 手写promise
```javascript
class MyPromise {
// 构造方法
  constructor(executor) {
    // 初始化this指向
    this.initBind();
    // 初始化值
    this.initValue();
    // 执行传过来的函数 有 throw 相当于执行reject
    try {
	executor(this.resolve, this.reject);
    } catch (e) {
	// 捕捉到错误直接执行reject
	this.reject(e)
    }
}
initBind() {
    this.resolve = this.resolve.bind(this);
    this.reject = this.reject.bind(this);
}
initValue() {
    // 初始化值
    this.PromiseResult = null;// 终值
    this.PromiseState = 'pending' // 状态
    this.onFulfilledCallbacks = [] // 保存成功回调
    this.onRejectedCallbacks = [] // 保存失败回调
}
resolve(value) {
    // 状态一旦改变就无法修改
    if (this.PromiseState != 'pending') return;
    // 如果执行resolve，状态变为fulfilled
    this.PromiseState = 'fulfilled';
    this.PromiseResult = value;// 终值传递过来的值
    // 执行保存的成功回调
    while (this.onFulfilledCallbacks.length) {
	this.onFulfilledCallbacks.shift()(this.PromiseResult)
    }
}
reject(reason) {
    if (this.PromiseState != 'pending') return;
    // 如果执行resolve，状态变为fulfilled
    this.PromiseState = 'rejected';
    this.PromiseResult = reason;
    // 执行保存的失败回调
    while (this.onRejectedCallbacks.length) {
	this.onRejectedCallbacks.shift()(this.PromiseResult)
    }
}
// 接收两个回调 onFulfilled, onRejected     回调要等状态确认后再执行，用数据save回调
then(onFulfilled, onRejected) {
    // 参数校验，一定要是函数
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : val => val
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason }
    // then支持链式调用 then执行后返回一个promise
    var thenPromise = new MyPromise((resolve, reject) => {
	// then是微任务 让resolvePromise函数异步执行
	const resolvePromise = callback => {
	    try {
		// 无返回为 undefined
		const x = callback(this.PromiseResult)
		if (x != null && x === thenPromise) {
		    // 不能返回自身哦
		    throw new Error('不能返回自身。。。')
		}
		if (x instanceof MyPromise) {
		    // 如果返回是promise对象，谁知道返回的promise是失败成功？只有then知道
		    x.then(resolve, reject)
		} else {
		    // 非promise直接成功
		    resolve(x)
		}
	    } catch (err) {
		// 处理报错
		reject(err)
		throw new Error(err)
	    }
	}
	if (this.PromiseState === 'fulfilled') {
	    // 如果当前为成功状态，执行第一个回调
	    resolvePromise(onFulfilled)
	} else if (this.PromiseState === 'rejected') {
	    resolvePromise(onRejected)
	} else if (this.PromiseState === 'pending') {
	    // 此时保存回调
	    this.onFulfilledCallbacks.push(resolvePromise.bind(this, onFulfilled))
	    this.onRejectedCallbacks.push(resolvePromise.bind(this, onRejected))
	}
    })
    // 返回这个包装的Promise
    return thenPromise
}
// 接收一个Promise数组，数组中如有非Promise项，则此项当做成功   如果所有Promise都成功，则返回成功结果数组，否则返回这个失败结果
static all(promises) {
    const result = []
    let count = 0
    return new MyPromise((resolve, reject) => {
	const addData = (index, value) => {
	    result[index] = value
	    count++
	    if (count === promises.length) resolve(result)
	}
	promises.forEach((item, index) => {
	    if (item instanceof MyPromise) {
		item.then(res => {
		    addData(index, res)
		}, err => reject(err))
	    } else {
		addData(index, item)
	    }
	})
    })
}
// 接收一个Promise数组，数组中如有非Promise项，则此项当做成功,哪个Promise最快得到结果，就返回那个结果，无论成功失败
static race(promises) {
    return new MyPromise((resolve, reject) => {
	promises.forEach(promise => {
	    if (promise instanceof MyPromise) {
		promise.then(res => {
		    resolve(res)
		}, err => {
		    reject(err)
		})
	    } else {
		resolve(promise)
	    }
	})
    })
}
}
```
### 使用Promise实现每隔1秒输出1,2,3
```javascript
//reduce(function(previousValue, currentValue, currentIndex, array) { /* ... */ }, initialValue)
//previousValue: 在第一次调用时，若指定了初始值 initialValue，其值则为 initialValue
//currentValue：数组中正在处理的元素。在第一次调用时，若指定了初始值 initialValue，其值则为数组索引为 0 的元素 array[0]，否则为 array[1]
const arr = [1, 2, 3];
arr.reduce((pre, cur)=>{
	return pre.then( () =>{
		return new Promise(r =>{
			setTimeout(() => r(console.log(cur)), 1000);
		})
	})
},Promise.resolve())
```

### 使用Promise实现红绿灯交替重复亮
```javascript
function red() {
  console.log("red");
}
function green() {
  console.log("green");
}
function yellow() {
  console.log("yellow");
}
const light = function (timer, cb) {
  return new Promise(resolve => {
    setTimeout(() => {
      cb()
      resolve()
    }, timer)
  })
}
const step = function () {
  Promise.resolve().then(() => {
    return light(3000, red)
  }).then(() => {
    return light(2000, green)
  }).then(() => {
    return light(1000, yellow)
  }).then(() => {
    return step()
  })
}

step();
```

### 
