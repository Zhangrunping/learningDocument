# JavaScript 的基本数据类型和应用类型

ECMASciprt 包括两个不同类型的值：基础数据类型和应用数据类型。  
基本数据类型指的是简单的数据段，引用数据类型指的是有多个值构成的对象。  
当我们把变量赋值给一个变量时，解析器首先要确认的就是这个值是基本类型值还是引用类型值。

## 常见的基本数据类型

按值访问，可操作保存在变量中实际的值  
值被保存在 栈内存 中，占据固定大小的空间  
number、boolean、string、null、undefined、bigInt、symbol

```
示例：
var a = 10;
var b = a;
b = 20;
console.log(a); // 值:10
```

示列代码中，b 获取的是 a 值的一份拷贝，虽然两个变量的值相等，但是这两个变量分别保存了不同的基本类型值；b 只是保存了 a 复制的一个副本，所以 b 的改变对 a 没有影响。

###### 示例代码赋值过程图

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/stack.png)

## 引用类型数据

对象类型 Object Type，例如：Object、Array、Function、Date 等。  
引用类型的值是按引用访问的  
保存在堆内存中的对象，不能直接访问操作对象的内存空间

```
示列：
var obj1 = { name: '张三' };
var obj2 = obj1;
obj2.name ='李二';
console.log(obj1); //李二
```

示列代码中，这两个引用数据类型都指向了同一个堆内存对象，‘obj1’赋值给‘obj2’，就是 obj1 把栈内存的引用地址复制了一份给 obj2；

###### 示例代码赋值过程图

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/heap.png)

## 判断数据类型方法

<h4 style="color:#ff502c">typeof</h4>

```
typeof '5' // string
typeof 5 // number
typeof null // object
typeof undefined // undefined
typeof true // boolean
typeof Symbol('5') // symbol
typeof 5n // bigint
typeof new Object(); // object
typeof new Function(); // function
```

<h4 style="color:#ff502c">instanceof</h4>

```
[] instanceof Array; // true
[] instanceof Object; // true

function Person() {};
const person = new Person();

person instanceof Person; // true
person instanceof Object; // true
```

<h4 style="color:#ff502c">constructor</h4>

```
'5'.__proto__.constructor === String // true
[5].__proto__.constructor === Array // true

undefined.**proto**.constructor // Cannot read property '**proto**' of undefined

null.**proto**.constructor // Cannot read property '**proto**' of undefined
```

<h4 style="color:#ff502c">toString</h4>

```
Object.prototype.toString.call('5') // [object String]
Object.prototype.toString.call(5) // [object Number]
Object.prototype.toString.call([5]) // [object Array]
Object.prototype.toString.call(true) // [object Boolean]
Object.prototype.toString.call(undefined) // [object Undefined]
Object.prototype.toString.call(null) // [object Null]
Object.prototype.toString.call(new Function()); // [object Function]
Object.prototype.toString.call(new Date()); // [object Date]
Object.prototype.toString.call(new RegExp()); // [object RegExp]
Object.prototype.toString.call(new Error()); // [object Error]
```

<h4 style="color:#ff502c">总结</h4>

typeof：适合基础类型和 function 类型的检测，无法判断 null 和 object。  
instanceof：适合自定义对象，也可以用来检测原生对象，在不同的 iframe 和 window 间检测时失效，注意 Object.create(null)对象的问题。  
constructor：基本能判断所有类型，除了 null 和 undefined，但是 constructor 容易被修改，也不能跨 ififrame 使用。  
toString：能判断所有类型，可将其封装成 Data Type()判断所有的类型。
