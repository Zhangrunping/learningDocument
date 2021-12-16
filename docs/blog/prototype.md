## 理解 JavaScript 中的原型与原型链

## 引用类型的四个规则

引用类型：Object、Function、Array、Date、RegExp

- 1、引用类型，都具有对象特性，即可自由扩展属性
- 2、引用类型，都有一个隐式原型[[Prototype]]的属性，属性值是一个普通的对象
- 3、引用类型，隐式原型[[Prototype]]的属性值指向它的构造函数的显式原型 prototype 属性值
- 4、当你试图得到一个对象的某个属性时，如果这个对象本身没有这个属性，那么它会去它的隐式原型[[Prototype]]（也就是它的构造函数的显式原型 prototype）中寻找

##### 规则一 ———— 引用类型，都具有对象特性，即可自由拓展属性

```
const arr = []
const obj = {}
const fn = function (){}
arr.a = 1
obj.a = 1
fn.a = 1
console.log(arr.a) // 1
console.log(obj.a) // 2
console.log(fn.a)  // 3
```

##### 规则二 ———— 引用类型，都有一个隐式原型[[Prototype]]的属性，属性值是一个普通的对象

```
const obj = {}
const arr = []
const fn = function (){}
console.log('obj.__proto__', obj.__proto__)
console.log('arr.__proto__', arr.__proto__)
console.log('fn.__proto__', fn.__proto__)
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/stack.png)

##### 规则三 ———— 引用类型，隐式原型[[Prototype]]的属性值指向它的构造函数的显式原型 prototype 属性值

```
const obj = {}
const arr = []
const fn = function (){}
console.log(obj.__proto__ === Object.prototype)
console.log(arr.__proto__ === Array.prototype)
console.log(fn.__proto__ === Function.prototype)
```

##### 规则四 ———— 当你试图得到一个对象的某个属性值，如果这个对象本身没有这个属性，那么它会去它的隐式原型上[[Prototype]]（也就是它的构造函数的显式原型）中寻找

```
const obj = { a:1 }
obj.toString() // ƒ toString() { [native code] }
```

首先，obj 对象并没有 toString 属性，之所以能获取到 toString 属性，是遵循了第四条规则，从它的构造函数 Object 的 prototype 里去获取

## 一个特例

```
function Person(name){
  this.name = name;
  return this; //这行可以不写，默认返回this对象
}
const nick = new Person('nick')
nick.toString() // ƒ toString() { [native code] }
```

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/stack.png)

按理说，nick 是 Person 构造函数生成的实例，而 Person 的 prototype 并没有 toString 方法，那么为什么 nick 实例能获取到 toString 方法呢？  
这里就引出原型链的概念了，nick 实例先从自身出发查找，并没有 toString 方法。找不到就往上继续查找，找到 Person 构造函数的 prototype 属性，还没有找到。
构造函数的 prototype 也是一个对象嘛，那对象的构造函数就是 Object，所以就找到了 Object.prototype 下的 toString 方法。

## 一张图片

用图片描述原型链
![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/stack.png)
最后一个 null，设计上是为了避免死循环设置的，Object.prototype 的隐式原型指向 null

## 一个方法

instanceof 运算符用于测试构造函数的 prototype 属性是否出现在对象原型链中的任何位置。  
instanceof 的简易手写版如下

```
// 变量R的原型 存在于 变量L的原型链上
function instance_of(L,R){
  const baseType = ['string', 'number', 'boolean', 'undefined', 'symbol']
  if(baseType.includes(typeof L)) return false;
  let RP = R.prototype;
  L = L.__proto__;
  while(true){
    if(L === null) return false // 找到最顶层
    if(L === RP) return true // 严格相等
    L = L.__proto__ // 没找到继续向上一层原型链查找
  }
}
```

再看下面这段代码代码

```
function Foo(name){
  this.name = name;
}
const f = new Foo('zhangsan');

f instanceof Foo // true
f instanceof Object // true
```

上面代码判断流程大致描述：

- 1、f instanceof Foo：f 的隐式原型[[Prototype]]和 Foo.prototype 是相等的，返回 true
- 2、f instanceof Object: f 的隐式原型[[Prototype]]和 Object.prototype 不想等，然后继续往上走。f 的隐式原型[[Prototype]]指向 Foo.prototype，所以继续用 Foo.prototype.**proto** 去对比 Object.prototype ，这会儿就相等了，因为 Foo.prototype 就是一个普通的对象。
