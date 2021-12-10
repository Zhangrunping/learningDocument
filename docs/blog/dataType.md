# JavaScript 的基本数据类型和应用类型

ECMASciprt 包括两个不同类型的值：基础数据类型和应用数据类型  
基本数据类型指的是简单的数据段，引用数据类型指的是有多个值构成的对象  
当我们把变量赋值给一个变量时，解析器首先要确认的就是这个值是基本类型值还是引用类型值

## 常见的基本数据类型

按值访问，可操作保存在变量中实际的值  
值被保存在 栈内存 中，占据固定大小的空间  
**number、boolean、string、null、undefined、bigInt、symbol**

```
示例：
var a = 10;
var b = a;
b = 20;
console.log(a); // 值:10
```

示列代码中，b 获取的是 a 值的一份拷贝，虽然两个变量的值相等，但是这两个变量分别保存了不同的基本类型值；b 只是保存了 a 复制的一个副本，所以 b 的改变对 a 没有影响

###### 示例代码赋值过程图

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/stack.png)

## 引用类型数据

对象类型 **Object**，例如：Object、Array、Function、Date 等  
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

**typeof**：适合基础类型和 function 类型的检测，无法判断 null 和 object  
**instanceof**：适合自定义对象，也可以用来检测原生对象，在不同的 iframe 和 window 间检测时失效，注意 Object.create(null)对象的问题  
**constructor**：基本能判断所有类型，除了 null 和 undefined，但是 constructor 容易被修改，也不能跨 ififrame 使用  
**toString**：能判断所有类型，可将其封装成 Data Type()判断所有的类型

## 数据类型转换

类型转换可以分为两种：**隐式类型转换**和**显式类型转换**  
**显式类型强制转换**是指当开发人员通过编写适当的代码用于在类型之间进行转换，比如：Number(value)  
**隐式类型转换**是指在对不同类型的值使用运算符时，值可以在类型之间自动的转换，比如：1 == null

##### JavaScript 中只有 3 钟类型的转换

类型转换的逻辑无论在原始类型或对象类型上，它们都只会转化为下面三种类型之一

- 转化为 Number 类型：Number()/parseFloat()/parseInt()
- 转化为 String 类型：String()/toString()
- 转化为 Boolean 类型：Boolean()

<h4 style="color:#ff502c">转换为boolean</h4>

显示：Number()方法可以用来显示将值转换为数字类型  
隐式：隐式类型转化通常在逻辑判断或着有逻辑运算符时被触发(||&&!)  
boolean 类型只会转为 true 和 false 两种结果。除了 0/NaN/空字符串/null/undefined 五个值为 false，其余都是 true

```
Boolean(2) // 显示类型转换
if(2){}    // 逻辑判断触发隐式类型转化
!!2        // 逻辑运算符触发隐式类型转换
2 || true  // 逻辑运算符触发隐式类型转换
```

逻辑运算符(比如||和&&)是在内部做了 boolean 类型转换，但实际上返回的是原始操作数的值，即使它们都不是 boolean 类型

```
// 返回number类型123，而不是boolean类型true
// 'hello'和123 仍然在内部会转换成boolean类型来计算表达式
let x = 'hello' && 123; // x === 123
```

<h4 style="color:#ff502c">转换为string</h4>

显示：String()方法可以用来显示将值转换为字符串  
隐式：隐式转换通常在有+运算符并且有一个是 string 类型时被触发

```
String([1,2,3]) // '1,2,3' 显示类型转换
String({})      // '[object Object]' //显示类型转换
1 + '123'       // '1123' //隐式类型转换
1 + {}          // '1[object Object]' //隐式类型转换
```

<h4 style="color:#ff502c">转换为number</h4>

显示：Number()方法可以用来显示将值转换为数字类型  
隐式：number 的隐式类型转换比较复杂，可以在下面多种情况被触发

- 比较操作（>, <, <=, >=）
- 按未操作（|, &, ^, ~）
- 算数操作（-, +, \* ,/ %） 当 + 操作存在任意的操作数是 string 类型时，不会触发 number 的隐式类型转换
- 一元 + 操作

```
// 字符串转换为数字：空字符串变为0，如果出现任意一个非有效数字字符，结果都是NaN
Number('')       // 0
Number('10px')   // NaN
Number('10')     // 10

// 布尔转换为数字
Number(true)     // 1
Number(false)    // 0

// null和undefined转换为数字
Number(null)       // 0
Number(undefined)  // NaN

// Symbol无法转为数字，会报错
Number(Symbol())   // Uncaught TypeError: Cannot convert a Symbol value to a number

// BigInt去除n
Number(12312412321312312n) //12312412321312312

// 对象转换为数字，会按照下面的步骤执行
1、先调用对象的Symbol.toPrimitive 这个方法，如果不存在这个方法
2、再调用对象的valueOf获取原始值，如果获取的值不是原始值
3、再调用对象的toString 把其变为字符串
4、最后再把字符串基础Number()方法转换为数字

let obj = { name: 'xxx' }
console.log(obj-10) //数字运算：先把obj转换为数字，在进行运算
//运行机制
obj[Symbol.toPrimitive]   // undefined
obj.valueOf               // {name: 'xxx'}
obj.toString              // [object Object]
Number('[object Object]') // NaN
NaN -10                   // NaN

```

##### 操作符==两边的隐式转换规则

如果两边数据类型不同，需要先转换为相同类型，然后在进行比较，以下情况需要注意

<h4 style="color:#ff502c">对象==字符串</h4>

```
[1,2,3] == '1,2,3'  // true
[1,2,3][Symbol.toPrimitive]   //undefined
[1,2,3].valueOf // [1,2,3]
[1,2,3].toString() // '1,2,3'
```

<h4 style="color:#ff502c">null/undefined</h4>

```
null == undefined   // true
null === undefined  // false
// null和undefined和其它任何值都不相等
```

<h4 style="color:#ff502c">对象==对象</h4>

```
对象与对象比较的是堆内存中的地址，地址相同则相等
{} == {} // false
```
