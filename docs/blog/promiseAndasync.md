#### 异步解决方案
- Promise对象
- Generator
- Async/await

#### Promise的理解

Promise对象是一个代理对象，它接收你传入的executor（执行器）作为入参，允许你把异步任务的成功任务和失败任务分别绑定在对应的处理方法上。一个Promise有三种状态：
- pending状态，表示进行中。这是Promise实例创建后的一个初始状态
- fulfilled状态，表示成功完成。这是在执行器中调用resolve后，达成的状态
- rejected状态，表示操作失败，被拒接。这是执行器中调用reject后，达成的状态

Promise实例的状态是可以改变的，但它只允许被改变一次。当我们的实例状态从pending切换至rejected后，就无法再扭转为fulfilled，反之同理。当Promise的状态为resolved时，会触发其对应的then方法入参的onfulfilled函数；当Promise的状态为rejected时会触发其对应的then方法入参里的onrejected函数。  
Promise的出现是为了解决回调地狱，一种既能实现异步又可以确保我们的代码好写又好看的解决方案

#### Promise的常见方法
> Promise.all(iterable)  

这个方法返回一个新的promise对象，该promise对象在iterable参数对象里，当所有的promise对象都成功的时候才会触发成功，一旦有任何一个iterable里面的promise对象失效则立即触发该promise对象的失败

```
let p1 = Promise.resolve('一号')
let p2 = '二号'
let p3 = new Promise((resolve, reject) => {
  setTimeout(resolve, 100, '三号')
})

Promise.all([p1,p2,p3]).then(values => {
  console.log(values) // ['一号', '二号', '三号']
})
```

> Promise.race(iterable)

当iterable参数里的任意一个子promise被成功或失败后，父promise马上也会用子promise的成功返回值或失败详情作为参数调用父promise绑定的相应函数，并返回改promise对象

```
let p1 = new Promise(function (resolve, reject){
  setTimeout(resolve, 100, '1号')
})

let p2 = new Promise(function (resolve, reject){
  setTimeout(resolve, 50, '二号')
})

// 这里因为2号返回得更早，所以返回值以2号为准
Promise.race([p1, p2]).then(function (value){
  console.log(value); // '二号'
})
```

> Promise.reject(reason)

返回一个失败为失败的promise对象，并将给定的的失败信息传递给对应的处理方法

> Promise.resolve(value)

返回一个promise对象，但这个对象的状态是由自己传入的value觉得的，情形分为以下两种
- 如果传入的是一个带then方法的对象(thenable对象)，返回Promise对象的最终状态由then方法执行决定的
- 否则的话，返回的Promise对象状态为fulfilled，同时这里的value会作为then方法中指定的onfulfilled的入参

#### 命题一

```
const promise = new Promise((resolve, reject) => {
  console.log(1)
  resolve()
  console.log(2);
})

promise.then(() => {
  console.log(3);
})

console.log(4);
```

输出：1、2、4、3；then方法中传入的任务是一个异步任务，resolve()这个调用，作用是将Promise的状态从pending置为fulfilled，这个新状态会让Promise知道'我的then方法中的那个任务可以执行了'————注意是'可以执行了'，而不是'立即就会执行'。毕竟作为一个异步任务，需要先将同步代码任务执行完之后再执行，所以3排在了最后;

#### 命题二

```
const promise =  new Promise((resolve, reject) => {
  resolve('第一次 resolve')
  console.log('resolve后的普通逻辑');
  reject('error')
  resolve('第二次 resolve')
})

promise.then((res) => {
  console.log('then:', res);
}).catch((err) => {
  console.log(err);
})

// 输出：
// resolve后的普通逻辑
// 第 1 次 resolve
```

在这段代码中，promise对象初始状态为pending，在函数体第一行中调用了resolve方法把promise状态置为fulfilled状态。这个切换之后，后续所有尝试进一步作状态切换的动作全部不生效，所以后续的reject、resolve方法忽略就好

#### 命题三

```
Promise.resolve(1)
.then(Promise.resolve(2))
.then(3)
.then()
.then(console.log) // 1
```
在这段代码中，最后输入1；在then方法里允许我们传入两个参数：onFulfilled（成功态的处理函数）和onRejected（失败态的处理函数）。  
可以两者都传，也可以只传入前者或者后者。但无论如何，then方法的入参只能是函数。如果塞入乱七八糟的东西，它会'翻脸不认人'  
代码执行具体过程：第一个then中传入的是一个Promise对象，then说不认识；第二个then中传入的是一个数字，then说不认识；第三个then中什么都没有，then说入参undefined不接受，直到最后一个入参，一个函数被传入进来，then说可以处理了；最后一个入参生效了


#### 模拟Promise

```
function CutePromise (executor){
  // value 记录异步任务成功的执行结果
  this.value = null
  // reason 记录异步任务失败的原因
  this.reason = null
  // status 记录当前状态，初始化是pending
  this.status = 'pending'

  // 缓存两个队列， 维护resolved和rejected各自对应的处理函数
  this.onResolvedQueue = [];
  this.onRejectedQueue = [];

  // 保存this
  var self = this;

  // 定义 resolve 函数
  function resolve(value){
    // 如果不是pending状态，直接返回
    if(self.status !== 'pending') return;

    // 异步任务成功，把结果赋值给value
    self.value = value
    // 当前状态切换为 resolved
    self.status = 'resolved'
    // 用setTimeout 延迟队列任务的执行
    setTimeout(function (){
      self.onResolvedQueue.forEach(resolved => resolved(self.value))
    })
  }

  // 定义 reject 函数
  function reject(reason){
    // 如果不是pending状态，直接返回
    if(self.status !== 'pending') return;
    // 异步任务失败，把结果赋值给reason
    self.reason = reason
    // 当前状态切换为 reason
    self.status = 'rejected'
    // 用setTimeout 延迟队列任务的执行
    setTimeout(function (){
      // 批量执行 rejected队列里的任务
      self.onRejectedQueue.forEach(rejected => rejected(self.reason))
    })
  }

  // 把resolve和reject能力赋予执行器
  executor(resolve, reject)
}

CutePromise.prototype.then = function (onResolved, onRejected) {
  // 注意：onResolved和onRejected 必须是函数；如果不是，我们此处用一个透传来兜底
  if(typeof onResolved !== 'function') {
    onResolved = function (x) {return x}
  }

  if(typeof onRejected !== 'function') {
    onRejected = function (e) {throw e}
  }

  // 保存this
  var self = this;
  // 判断是否是resolved状态
  if(self.status === 'resolved') {
    // 如果是 执行对应的处理方法
    onResolved(self.value)
  }else if(self.status === 'rejected') {
    // 若是rejected状态，则执行rejected对应方法
    onRejected(self.reason)
  }else if(self.status === 'pending') {
    // 若是pending状态，则只对任务做入队处理
    self.onResolvedQueue.push(onResolved)
    self.onRejectedQueue.push(onRejected)
  }
  
  return this;
}
const cutePromise = new CutePromise(function (resolve, reject) {
    resolve('成了');
});
cutePromise.then((value) => {
    console.log(value)
    console.log('我是第 1 个任务')
}).then(value => {
    console.log('我是第 2 个任务')
});
// 依次输出“成了” “我是第 1 个任务” “我是第 2 个任务”
```

