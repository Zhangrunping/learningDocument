## 什么是 Promise，我们用 Promise 解决了什么

Promise 是异步编程的一种解决方案；从语法上讲，Promise 是一个对象，通过它可以获取到异步操作的消息；从本意上讲，他是承诺，承诺它过段时间给你一个结果。  
Promise 有三种状态：pending（等待状态），fulfilled（成功状态），rejected（失败状态）；状态一旦改变，就不会再变，创造 Promise 实例后，它会立即执行

Promise 是用来解决两个问题的：

- 1、回调地狱，代码难以维护，常常第一个的函数输出是第二个函数的输入这种现象
- 2、Promise 可以支持多个并发的请求，获取并发请求中的数据

## Promise 用法

Promise 是一个构造函数，有 all、reject、resolve 等方法，原型上有 then、catch 等方法

```
let p = new Promise((resolve,reject) => {
  // 做一些异步操作
  setTimeout(() => {
    resolve('this is resolve')
  },1000)
})
```

Promise 的构造函数接收一个参数：函数。并且这个函数需要传入两个参数：

- resolve：异步操作执行成功后的回调函数
- reject：异步操作执行失败后的回调函数

#### then 链式操作的方法

从表面上看，Promise 只是能够简化层层回调的写法，而实质上，Promise 的精髓是‘状态’，用维护状态、传递状态的方式来使得回调函数能够及时调用，比传递 callback 函数要简单
灵活得多。

```
p.then((data)=>{
  console.log(data)
}).then((data)=>{
  console.log(data)
}).then((data)=>{
  console.log(data)
})
```

#### reject 的用法

把 Promise 的状态设置为 rejected，在 then 中就能捕捉到，然后执行'失败'情况的情况，看下面代码

then 方法可以接受两个参数，第一个对应 resolve 的回调，第二个对应 reject 的回调

```
  let p = new Promise((resolve, reject) => {
    // 做一些异步操作
    setTimeout(function (){
      var num = Math.ceil(Math.random() * 10)
      if(num >= 5) {
        resolve(num)
      }else{
        reject('数字太大了')
      }
    },1000)
  })

  p.then((data) => {
    console.log('resolved', data)
  },(err) => {
    console.log('rejected', err)
  })
```

#### catch 的用法

catch 方法是用来指定 reject 的回调

```
p.then((data) => {
    console.log('resolved',data);
}).catch((err) => {
    console.log('rejected',err);
});
```

还有一个作用就是如果抛出异常了（代码出错），那么并不会报错卡死 js，而是进入 catch 方法中

```
p.then((data) => {
    console.log('resolved',data);
    console.log(somedata); //此处的somedata未定义
})
.catch((err) => {
    console.log('rejected',err);
});
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async1.png)

#### all 用法

谁最后执行完成，就以谁为准执行回调，all 接收一个数组参数，里面的值最终都算返回 Promise 对象  
Promise.all()方法提供了并行执行异步操作的能力，在所有异步操作执行完成后才执行回调

```
let Promise1 = new Promise(function(resolve, reject){})
let Promise2 = new Promise(function(resolve, reject){})
let Promise3 = new Promise(function(resolve, reject){})

let p = Promise.all([Promise1, Promise2, Promise3])

p.then(funciton(){
  // 三个都成功则成功
}, function(){
  // 只要有失败，则失败
})

```

#### race 方法

谁最先执行完成，就以谁为准执行回调。使用场景：比如用 race 给某个异步请求设置请求超时，并且在超时后执行相关的操作，代码如下

```
// 请求某个图片资源
function requesImg(){
  return new Promise ((resolve, reject)=>{
    let img = document.createElement('img');
    img.onload = function (){
      resolve(img)
    }
    img.src = '图片的路径'
  })
}

// 延迟函数，用于给请求计算
function timeout(){
  return new Promise((resovle, reject)=>{
    setTimeout(() => {
      reject('图片请求超时')
    }, 3000);
  })
}

Promise.race([requesImg(),timeout()]).then((data)=>{
  console.log('data',data);
}).catch((err)=>{
  console.log('err',err);
})
```

requesImg 函数会异步请求一张图片，把地址写为‘图片的路径’，所以肯定是无法请求到的  
timeout 函数是一个延时 3 秒的异步操作。把两个返回 Promise 对象的函数放入 race，它们就会比谁更快。如果 3 秒之内图片请求成功了，那么便会进入 then 方法。如果 3 秒内还未
成功返回，那么就会触发 timeout 函数中的 reject 回调。则进入 catch。抛出‘图片请求超时’的信息

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async2.png)

## 什么是 async

async 函数是 Generator 函数的语法糖，使用关键字 async 来表示，在函数内部使用 await 来表示异步。相比 Generator，async 函数的改进在于下面 4 点

- 1、内置执行器，Generator 函数执行必须依靠执行器，而 async 函数自带执行器，调用方式和普通函数调用方式一行
- 2、更好的语义，async 和 await 相较于\*与 yield 更加语义化
- 3、更广的适用性， co 模块约定，yield 命令后面只能是 Thunk 函数或 Promise 对象，而 async 函数的 await 命令后面则可以是 Promise 或者原始类型的值（string、boolean、number）
  但这是等同与同步操作
- 4、 返回值是 Promise，async 函数返回值是 Promise 对象，比 Generator 函数返回的 lterator 对象方便，可以直接使用 then()方法调用

async 表示函数里有异步操作  
await 表示紧跟在后面的表达式需要等待结果

## async 怎么用

```
async function asyncFu(){
  return 'hello world'
}

asyncFu()
```

这样就表示这是异步函数，返回的结果
![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async3.png)

返回的一个 Promise 对象，状态为 resolved，参数是 return 的值。

```
async function asyncFn() {
    return '我后执行'
}
asyncFn().then(result => {
    console.log(result);
})
console.log('我先执行');
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async4.png)

上面执行代码结果先打印‘我先执行’，虽然是 asyncFn()先执行，但是已经被定义为异步函数了，不会影响后续代码的执行。  
async 定义的函数内部默认返回一个 Promise 对象，如果函数内部抛出异常或者是返回 reject，都会使函数的 promise 状态为失败 reject

```
async function e(){
  throw new Error('has Error')
}

e().then(success => console.log('成功', success))
  .catch(error => console.log('失败', error))
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async5.png)

上面代码看到函数内部抛出一个异常，返回 reject，async 函数接收到之后，判定执行失败进入 catch，该返回的错误打印出来

```
async function throwStatus() {
  return '可以返回所有类型的值'
}

throwStatus().then(success => console.log('成功', success))
  .catch(error => console.log('失败', error))
  // 打印结果
  // 可以返回所有类型的值
```

上面代码，async 函数接收返回的值，发现不是异常或者 reject，则判定成功。这里可以 return 各种数据类型的值，false，undefined，NaN...总之，都是 resolve  
但是返回如下结果会使 async 函数判定 reject  
1、内部含有直接使用并且未申明的变量或者函数  
2、内部抛出一个错误（throw new Error） 或者返回 reject 状态（return Promise.reject('执行失败')）  
3、函数方法执行错误(Object 使用 push 方法)等待....
还有一点，在 async 里，必须将结果 return 回来，不然的话不管执行 reject 或者 resolved 的值都为 undefined，建议使用箭头函数  
其余返回结果都是判定 resolved 成功执行

```
// 正确的reject方法，必须将reject状态return出去
async function PromiseError(){
  return Promise.reject('has Promise reject')
}

// 这是错误的做法，并且判定resolve，返回值undefined并且Uncaught报错
async function PromiseError(){
  Promie.reject('这是错误的做法')
}
PromiseError.then(success => console.log('成功', success))
              .catch(error => console.log('失败', error));
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async6.png)

## await 是什么

await 意思是 async await（异步等待）。这个关键字只能在 async 定义的函数里使用。任何 async 函数都会默认返回 Promise，并且这个 promise 解析的值都将会
是这个函数的返回值，而 async 函数必须等到内部所有的 await 命令的 proimse 对象执行完，才会发生状态改变

```
async function awaitReturn(){
  return await 1
}
awaitReturn().then(success => console.log('成功',success))
            .catch(error => consolo.log('失败',error))
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/async7.png)
在上面代码函数里，有一个 async 函数，async 函数会等到 await 1 这一步执行完了才会返回 promise 状态，判定为 resolved

**很多人认为 await 会一直等待之后的表达式执行完之后才会继续执行后面的代码，实际上 await 是一个让出线程的标志。await 后面的函数会先执行一遍**  
**比如（await fn()的 fn，并非是下一行代码），然后就会跳出整个 async 函数来执行后面 js 执行栈的代码。等本轮事件循环执行完了之后又会调到 async 函数总等待**
**async 后面的表达式的返回值，如果返回值为非 promise 则继续 async 函数后面的代码，否则将返回的 promise 放入 promise 队列（Promise 的 Job Queue）**

```
// 一个简单的例子
```
