#### 数据类型判断
typeof和instanceof方法判断类型都会使判断类型不准确，但是可以通过Object.prototype.toString实现

```
  function typeOf(source){
    return Object.prototype.toString.call(source).slice(8, -1).toLowerCase()
  }

  console.log(typeOf(1));  // number
  console.log(typeOf([])); // array
  console.log(typeOf({})); // object
  console.log(typeOf(new Date())); // date
  console.log(typeOf(true)); // boolean
```

#### 继承
  > 1、原型链继承
  
```
  function Animal(){
    this.colors = ['black', 'white']
  }

  Animal.prototype.getColor = function (){
    return this.colors
  }

  function Dog(){}
  Dog.prototype = new Animal();

  let dog1 = new Dog();
  dog1.colors.push('brown')
  let dog2 = new Dog();
  
  console.log(dog2.getColor())
```

原型链继承存在问题：
  - 问题1：原型中包含的引用类型属性将被所有实例共享
  - 问题2：子类在实例化的时候不能给父类构造函数传参  
  

> 2、借用构造函数实现继承

```
  function Animal(name){
    this.name = name;
    this.getName = function (){
      return this.name
    }
  }

  function Dog(name){
    Animal.call(this, name)
  }

  Dog.prototype = new Animal();
```

构造函数实现继承解决了引用类型共享以及传参问题。但方法必须定义在构造函数中，会导致每次创建子类实例都会创建一遍方法

> 3、组合继承
 
组合继承结合了原型链和构造函数，将两者的优点集中起来，基本的思路是使用原型链继承原型上的属性和方法，而通过构造函数继承实例属性，这样既可以把方法定义在原型上以
实现复用，又可以让每个实例都有自己的属性

```
  function Animal (name){
    this.name = name;
    this.colors = ['black', 'while']
  }

  Animal.prototype.getName = function (){
    return this.name;
  }

  function Dog(name, age){
    Animal.call(this, name)
    this.age = age
  }

  Dog.prototype = new Animal();

  let dog1 = new Dog('苹果', 2)
  dog1.colors.push('brown')
  let dog2 = new Dog('香蕉', 1)
  console.log(dog2)   // { name: "香蕉", colors: ["black", "white"], age: 1 }
```

> 4、寄生式组合继承

组合继承已经相对完善了，但还存在的一个问题就是调用了两次父类构造函数，第一次在new Animal()，第二次在Animal.call()  
解决方法就是不直接调用父类构造函数给子类原型赋值，而是通过创建空对象获取父类原型的副本  

```
  function Animal (name){
    this.name = name;
    this.colors = ['black', 'while']
  }

  Animal.prototype.getName = function (){
    return this.name;
  }

  function Dog(name, age){
    Animal.call(this, name)
    this.age = age
  }

  <!-- Dog.prototype = new Animal(); -->
  Dog.prototype = Object.create(Animal.prototype)
  Dog.prototype.constructor = Dog

  let dog1 = new Dog('苹果', 2)
  dog1.colors.push('brown')
  let dog2 = new Dog('香蕉', 1)
  console.log(dog2)   // { name: "香蕉", colors: ["black", "white"], age: 1 }
```

> 5、 class 实现继承

```
  class Animal {
    constructor(name){
      this.name = name
      this.colors = ['black', 'while'];
    }

    getName(){
      return this.name
    }
  }

  class Dog extends Animal {
    constructor(name, age){
      super(name)
      this.age = age
    }
  }
```

#### 数组去重
 
> ES5 实现：

```
  function unique(arr){
    var res = arr.filter(function (item, index, array){
      return array.indexOf(item) === index
    })
    return res
  }
```

> ES6 实现：

```
  function unique(arr){
    return [...new Set(arr)]
  } 
```

#### 数据对象去重

> filter和map 实现

```
  function uniqueFunc(arr, unId){
    const res = new Map();
    return arr.filter((item) => !res.has(item[unId]) && res.set(item[unId],'1'))
  }
```

> reduce 实现

```
  function uniqueFunc2(arr, uniId){
    let hash = {}
    return arr.reduce((accum,item) => {
      hash[item[uniId]] ? '' : hash[item[uniId]] = true && accum.push(item)
      return accum
    }, [])
  }
```

#### 数组扁平化
数组扁平化就是就是将[1, [2, [3 ]]]多层数组转化成[1, 2, 3]  
使用flat()方法可以直接将多层数组转化成一层 [1, [2, [3 ]]].flat(2)  // [1, 2, 3]

> ES5 实现

```
  function flatten(arr){
    let result = [];
    for(let i = 0, len = arr.length; i < len; i++) {
      console.log(i);
      if(Array.isArray(arr[i])){
        result = result.concat(flatten(arr[i]))
      }else{
        result.push(arr[i])
      }
    }
    return result
  }
```

> ES6 实现

```
  function flatten(arr){
    while (arr.some(item => Array.isArray(item))){
      arr = [].concat(...arr)
    }
    return arr
  }
```

#### 深浅拷贝

> 浅拷贝 只考虑对象类型

```
  function shallowCopy(obj){
    if(typeof obj !== 'object') return;

    let newObj = obj instanceof Array ? [] :{};
    for(let key in obj) {
      if(obj.hasOwnProperty(key)) {
        newObj[key] = obj[key]
      }
    }
    return newObj;
  }
```

> 简单版浅拷贝 只考虑普通对象属性，不考虑内置对象和函数

```
  function deepClone(obj){
    if(typeof obj !== 'object') return;      
      let newObj = obj instanceof Array ? [] : {};
      for(let key in obj) {
        if(obj.hasOwnProperty(key)) {
          newObj[key] = typeof obj[key] === 'object' ? deepClone(obj[key]) : obj[key]
        }
      }
    return newObj;
  }
```

> 复杂版深克隆 在简单版的基础上，还考虑了内置对象比如Date、RegExp等对象和函数以及解决了循环引用的问题 

```
  const isObject = (target) =>  (typeof target === 'object' || typeof target === 'function') && target != null;

  function deepClone(target, map = new Map()){
    if(map.get(target)){
      return target
    }

    // 获取当前值的构造函数，获取它的类型
    let constructor = target.constructor;
    // 检测当前对象target是否与正则、日期格式对象匹配
    if(/^(RegExp|Date)$/i.test(constructor.name)){
      return new constructor(target)
    }

    if(isObject(target)){
      map.set(target, true) // 为循环引用的对象做标记
      const cloneTarget = Array.isArray(target) ? [] : {};
      for(let prop in target) {
        if(target.hasOwnProperty(prop)){
          cloneTarget[prop] = deepClone(target[prop])
        }
      }
      return cloneTarget;
    }else{
      return target;
    }
  }
```

#### 解析URL为对象

```
function parseParam(url) {
    const paramsStr = /.+\?(.+)$/.exec(url)[1]; // 将 ? 后面的字符串取出来
    const paramsArr = paramsStr.split('&'); // 将字符串以 & 分割后存到数组中
    let paramsObj = {};
    // 将 params 存到对象中
    paramsArr.forEach(param => {
        if (/=/.test(param)) { // 处理有 value 的参数
            let [key, val] = param.split('='); // 分割 key 和 value
            val = decodeURIComponent(val); // 解码
            val = /^\d+$/.test(val) ? parseFloat(val) : val; // 判断是否转为数字
    
            if (paramsObj.hasOwnProperty(key)) { // 如果对象有 key，则添加一个值
                paramsObj[key] = [].concat(paramsObj[key], val);
            } else { // 如果对象没有这个 key，创建 key 并设置值
                paramsObj[key] = val;
            }
        } else { // 处理没有 value 的参数
            paramsObj[param] = true;
        }
    })
    return paramsObj;
  }
```

#### 事件总线(发布订阅模式)

```
  class EventEmitter {
    constructor() {
      this.cache = {}
    }

    on(name, fn) {
      if (this.cache[name]) {
        this.cache[name].push(fn)
      } else {
        this.cache[name] = [fn]
      }

    }

    off(name, fn) {
      let tasks = this.cache[name]

      if (tasks) {
        const index = tasks.findIndex(f => f === fn || f.callback === fn)
        if (index >=0) {
          tasks.splice(index, 1) 
        }
      }
    }

    emit(name, once = false, ...args) {
      if (this.cache[name]) {
        // 创建副本，如果回调函数内继续注册相同事件，会造成死循环
        let tasks = this.cache[name].slice()
        
        for(let fn of tasks) {
          fn(...args)
        }
        
        if(once) delete this.cache[name]
      }
    }
  }

  let eventBus = new EventEmitter();

  let f1 = function (name, age) {
    console.log(`${name} ${age}`)
  }

  let f2 = function (name, age) {
    console.log(`hello ${name} ${age}`)
  }

  eventBus.on('log', f1)
  eventBus.on('log', f2)
  eventBus.emit('log', false, 'jeee', 18)
  // jeee 18
  // hello jeee 18
```

#### 防抖
触发高频事件N秒后只会执行一次，如果N秒内事件再次触发，则会重新计时

> 简单版：函数内部支持使用this和event对象

```
  function debounce(func, wait) {
    let timeout

    return function () {
      let context = this
      let args = arguments

      if(timeout) clearTimeout(timeout)

      timeout = setTimeout(function (){
        func.apply(context, args)
      }, wait)
    }
  }

  // 使用
  <!-- var node = document.getElementById('layout')
  function getUserAction(e) {
    console.log(this, e)  // 分别打印：node 这个节点 和 MouseEvent
    node.innerHTML = count++;
  };
  node.onmousemove = debounce(getUserAction, 1000) -->
```

> 最终版：除了支持this和event，还支持以下功能

- 支持立即执行
- 函数可能有返回值
- 支持取消功能

```
function debounce(func, wait, immediate) {
    var timeout, result;
    
    var debounced = function () {
        var context = this;
        var args = arguments;
        
        if (timeout) clearTimeout(timeout);
        if (immediate) {
            // 如果已经执行过，不再执行
            var callNow = !timeout;
            timeout = setTimeout(function(){
                timeout = null;
            }, wait)
            if (callNow) result = func.apply(context, args)
        } else {
            timeout = setTimeout(function(){
                func.apply(context, args)
            }, wait);
        }
        return result;
    };

    debounced.cancel = function() {
        clearTimeout(timeout);
        timeout = null;
    };

    return debounced;
  }

  // 使用
  <!-- var setUseAction = debounce(getUserAction, 10000, true);
  // 使用防抖
  node.onmousemove = setUseAction

  // 取消防抖
  setUseAction.cancel() -->
```

#### 节流
触发高频事件、且N秒内只执行一次

> 简单版：使用时间戳来实现，立即执行一次，然后每N秒执行一次

```
  function throttle(func, wait) {
      var context, args;
      var previous = 0;

      return function() {
          var now = +new Date();
          context = this;
          args = arguments;
          if (now - previous > wait) {
              func.apply(context, args);
              previous = now;
          }
      }
  }
```

> 最终版：支持取消节流；另外通过传入第三个参数，options.leading 来表示是否可以立即执行一次，opitons.trailing 表示结束调用的时候是否还要执行一次，默认都是 true。
注意设置的时候不能同时将 leading 或 trailing 设置为 false。

```
function throttle(func, wait, options) {
    var timeout, context, args, result;
    var previous = 0;
    if (!options) options = {};

    var later = function() {
        previous = options.leading === false ? 0 : new Date().getTime();
        timeout = null;
        func.apply(context, args);
        if (!timeout) context = args = null;
    };

    var throttled = function() {
        var now = new Date().getTime();
        if (!previous && options.leading === false) previous = now;
        var remaining = wait - (now - previous);
        context = this;
        args = arguments;
        if (remaining <= 0 || remaining > wait) {
            if (timeout) {
                clearTimeout(timeout);
                timeout = null;
            }
            previous = now;
            func.apply(context, args);
            if (!timeout) context = args = null;
        } else if (!timeout && options.trailing !== false) {
            timeout = setTimeout(later, remaining);
        }
    };
    
    throttled.cancel = function() {
        clearTimeout(timeout);
        previous = 0;
        timeout = null;
    }
    return throttled;
}
```

#### 函数柯里化
柯里化是一种将使用多个参数的一个函数转换成一些列使用一个参数的函数的技术

```
  function _curry(fn, len = fn.length) {
    return curry.call(this, fn, len)
  }

  function curry(fn, len, ...args){
    return function (...params){
      let _args = [...args, ...params]

      if(_args.length >= len) {
        return fn.apply(this, ...args)
      } else {
        return _curry.call(this,fn,len,..._args)
      }
    }
  }
```

#### JSONP
JSONP 核心原理：script标签不受同源策略约束，所以可以用来进行跨域请求，优点：兼容性好，缺点：只能用于get请求

```
  const jsonp = ({url, params, callbackName}) => {
    const generateUrl = () => {
      let dataSrc = ''
      for (let key in params) {
        if(params.hasOwnProperty(key)) {
          dataSrc += `${key}=${params[key]}&`
        }
      }
      dataSrc += `callback=${callbackName}`
      return `${url}?${dataSrc}`
    }

    return new Promise((resolve, reject) => {
      const scriptEle = document.createElement('script')
      scriptEle.src =  generateUrl()
      document.body.appendChild(scriptEle)
      window[callbackName] = data => {
        resolve(data)
        document.removeChild(scriptEle)
      }
    })
  }
```

#### AJAX

```
  const getJSON = function(url) {
    return new Promise((resolve, reject) => {
        const xhr = XMLHttpRequest ? new XMLHttpRequest() : new ActiveXObject('Mscrosoft.XMLHttp');
        xhr.open('GET', url, false);
        xhr.setRequestHeader('Accept', 'application/json');
        xhr.onreadystatechange = function() {
            if (xhr.readyState !== 4) return;
            if (xhr.status === 200 || xhr.status === 304) {
                resolve(xhr.responseText);
            } else {
                reject(new Error(xhr.responseText));
            }
        }
        xhr.send();
    })
  }

```

#### 实现函数原型方法

> call
使用一个指定的this值和一个或者多个参数来调用一个函数
- this可能传入null
- 传入不固定个数的参数
- 函数可能有返回值

```
  Function.prototype.myCall = function (context){
    context =  context || window
    context.fn = this;

    let args = [];
    for(let i = 1, len = arguments.length; i < len; i++) {
      args.push('arguments[' + i + ']')
    }

    console.log(arguments);

    let result = eval('context.fn('+ args +')')
    delete context.fn
    return result;
  }
```

> apply
apply与call一样，唯一的区别就是call是传入不固定个数的参数，而apply是传入一个数组
- this 可能传入null
- 传入一个数组
- 函数可能有返回值

```
  Function.prototype.myApply = function (context, arr){
    context = context || window
    context.fn = this;

    let result;
    if (!arr) {
      result = context.fn();
    } else {
      let args = [];
      for(let i = 0, len = arr.length; i < len; i++) {
        args.push('arr['+ i +']')
      }
      result = eval('context.fn('+ args +')')
    }

    delete context.fn;
    return result;
  }
```


> bind
bind方法会创建一个新函数，在bind()被调用时，这个新函数的this被指定为bind()的第一个参数，而其余参数将作为新函数的参数，仅调用时使用  
实现要点：
- bind()除了this外，还可传入多个参数
- bind创建的新函数的新函数可能传入多个参数
- 新函数可能被当做构造函数调用
- 函数可能有返回值

```
  Function.prototype.myBind = function (context){
    let self = this
    let args = Array.prototype.slice.call(arguments, 1)
    let fNOP = function (){}

    let fBound = function (){
      let bindArgs = Array.prototype.slice.call(arguments)
      return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs))
    }

    fNOP.prototype = fBound.prototype
    fBound.prototype = new fNOP()
    return fBound
  }
```

#### 实现new关键字 
nwe 运算符用来创建用户自定义的对象类型的实例或者具有构造函数的内置对象的实例  
实现要点：
- new 会产生一个新对象
- 新对象需要能够访问到构造函数的属性，所以需要重新指定它的原型
- 构造函数可能会显示返回

```
  function objectFactory() {
    var obj = new Object()
    Constructor = [].shift.call(arguments);
    obj.__proto__ = Constructor.prototype;
    var ret = Constructor.apply(obj, arguments);
    
    // ret || obj 这里这么写考虑了构造函数显示返回 null 的情况
    return typeof ret === 'object' ? ret || obj : obj;
  };
```

#### 实现instanceof关键字
instanceof是判断构造函数的prototype属性是否出现在实例上的原型链

```
  function instanceOf(left, right){
    let proto = left.__proto__
    while(true){
      if(proto === null) return false
      if(proto === right.prototype) {
        return true
      }
      proto = proto.__proto__ 
    }
  }
```

#### 实现Promise

```
  function CutePromise(executor){
    // value 记录异步任务成功的执行结果
    this.value = null
    // reason 记录异步任务失败的原因
    this.reason = null
    // status 记录当前状态，初始化时pending
    this.status = 'pending'

    //缓存两个队列，维护resolved和rejected各自对应的处理函数
    this.onRejectedQueue = [];
    this.onResolvedQueue = [];

    let self = this

    // 定义resolve 函数
    function resolve (value){
      // 如果不是pending状态 直接返回
      if(self.status !== 'pending') return;
      // 异步任务成功，把结果赋值给value
      self.value = value
      // 当前状态切换为resolved
      self.status = 'resolved'

      //用setTimeout 延迟队列任务的执行
      setTimeout(function (){
        self.onResolvedQueue.forEach(resolved => resolved (self.value))
      })
    }

    // 定义reject 函数
    function reject (reason){
      // 如果不是pending状态 直接返回
      if(self.status !== 'pending') return;
      
      // 异步任务失败，把结果赋给value
      self.reason = reason
      // 当前状态切换为rejected
      self.status = 'rejected'

      //用setTimeout 延迟队列任务的执行
      setTimeout(function(){
          // 批量执行 rejected 队列里的任务
          self.onRejectedQueue.forEach(rejected => rejected(self.reason));
      });
    }

    // 把resolve和 reject 能力赋予执行器
    executor(resolve, reject)

  }

  //then方法接收两个函数作为入参
  CutePromise.prototype.then = function (onResolved, onRejected){

    // 注意：onResolved和onRejected必须是函数；如果不是，我们此处用一个透传来兜底
    if(typeof onResolved !== 'function') {
      onResolved = function(x) {return x}
    }

    if(typeof onRejected !== 'function') {
      onRejected = function(e) {return e}
    }

    let self = this;
    // 判断是否是resolved状态
    if(self.status === 'resolved') {
      onResolved(self.value)
    }else if(self.status === 'rejected') {
      onRejected(self.reason)
    }else if(self.status === 'pending') {
      // 若是pending状态，则只对任务做入队处理
      self.onResolvedQueue.push(onResolved)
      self.onRejectedQueue.ppush(onRejected)
    }

    return this
  }
```





