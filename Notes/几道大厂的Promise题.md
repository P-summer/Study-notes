### 使用Promise实现每隔1秒输出1,2,3
```javascript
//reduce(function(previousValue, currentValue, currentIndex, array) { /* ... */ }, initialValue)
//在第一次调用时，若指定了初始值 initialValue，其值则为 initialValue
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
