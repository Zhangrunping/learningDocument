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
    console.log(`${name} ${age}`);
  }

  let f2 = function (name, age) {
    console.log(`hello ${name} ${age}`);
  }

  eventBus.on('log', f1)
  eventBus.on('log', f2)
  eventBus.emit('log', false, 'jeee', 18)
  // jeee 18
  // hello jeee 18
```

#### 图片懒加载


