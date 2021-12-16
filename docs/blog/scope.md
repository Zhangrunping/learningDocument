## 什么是作用域(Scope)

作用域就是一套规则，用于确定在何处以及如何查找变量（标识符）的规则
JavaScript 采用词法作用域（lexical scoping），也就是静态作用域

```
function fun(){
  var inVariable = '内层变量'
}
fun();
console.log(inVariable) // Uncaught ReferenceError: inVariable is not defined
```

从上面的例子中可以体会到作用域的概念，变量 inVariable 在全局作用域没有声明，所以在全局作用域下取值就会报错。我们可以这样理解：作用域就是一个独立的地盘，
让变量不会外泄、暴露出去。也就是说作用域最大的好处就是隔离变量，不同作用域下同名变量不会冲突

ES6 之前 JavaScript 没有块级作用域只有全局作用域和函数作用域。ES6 的到来提供了块级作用域，可通过新增命令 let 和 const 来体现

## 全局作用域和函数作用域

###### 全局作用域

在代码中任何地方都能访问到的对象全局作用域，一般来说以下情况拥有全局作用域

- 最外层函数和最外层函数外面定义的变量拥有全局作用域

```
var outVariable = '最外层变量'
function outFun(){ // 最外层函数
  var inVariable = '内层变量'
  function innerFun(){
    console.log(inVariable)
  }
  innerFun()
}
console.log(outVariable) // '最外层变量'
outFun()
console.log(inVariable) // inVariable is not defined
innerFun() // innerFun is not defined
```

- 所有未定义直接赋值的变量自动声明为拥有全局作用域

```
function outFun2(){
  variable = '未定义直接赋值的变量'
  var inVariable2 = '内层变量'
}
outFun2()
console.log(variable) // '未定义直接赋值的变量'
console.log(inVariable2) // inVariable2 is not defined
```

- 所有 window 对象的属性拥有全局作用域
  一般情况下，window 对象的内置属性都拥有全局作用域，例如：window.name、window.location、window.top 等  
   全局作用域有个弊端：比如我们写了很多行代码，变量定义没有用函数包裹，它们全部都会在全局作用域中。这样就会污染全局命名空间，容易引起命名冲突

```
// 张三代码中
var data = { x:1 }
// 李二代码中
var data = { b:true }
```

这就是 jQuery,Zepto 等库的源码中，所有的代码中都会放在(function(){})()中，因为放在里面的所有变量，都不会外泄和暴露，不会污染到外面，不会对其它的库或者 js 脚本
造成影响。这也是函数作用域的体现

###### 函数作用域

函数作用域是指声明在函数内部的变量，和全局作用域相反，局部作用域一般只能在固定的代码片段中可以访问到，最常见的例如函数内部

```
function doSomething(){
  var blogName='zhangsan'
  function innerSay(){
    alert(blogName)
  }
}
alert(blogName) // 脚本错误
innerSay() // 脚本错误
```

作用域是分层的，内层作用域可以访问外层作用域的变量，反正则不行。可以看一下例子

![](https://raw.githubusercontent.com/Zhangrunping/learningDocument/master/docs/image/stack.png)

最后输出 1,2,6

- 泡泡 1 是全局作用域，有标识符 foo
- 泡泡 2 作用域 foo，有标识符 a，bar，b
- 泡泡 3 作用域 bar，有标识符 c

**值得注意的是**：块语句（大括号‘{}’中的语句），如 if 和 switch 条件语句或 for 和 while 循环语句，不像函数，它们不会创建一个新的作用域。
在块语句中定义的变量将保留在它们已经存在的作用域中

## 块级作用域

块级作用域可通过新增命令 let 或 const 声明，所声明的变量在指定的作用域外无法被访问。块级作用域在如下情况下被创建：  
1：在一个函数内部  
2：在一个代码块(由一对花括号包裹)内部

let 声明的语法与 var 声明的语法一致。基本上可以用 let 来代替 var 进行声明变量，将会将变量的作用域限制在当前代码块中。块级作用域有以下特点：

- 声明变量不会提升到代码块顶部

let/const 声明并不会提升到当前代码块顶部，需要手动将 let/const 声明放置到顶部，以便让变量在整个代码块内部可以使用

```
function getValue(){
  if(condition) {
    let value = 'blue'
    return value
  }else {
    // value is not defined
    return null
  }
  // value is not defined
}
```

- 禁止重复声明

如果一个标识符已经在代码块内部被定义，那么在此代码块内使用一个标识符进行 let 声明就会导致抛出错误。例如

```
var count = 30
let count = 40 // Uncaught SyntaxError: Identifier 'count' has already been declared
```

在上面例子中，count 变量被申明了两次，一次使用 var，另一次使用 let。因此 let 不能在同一作用域内重复声明一个已有标识符，此处的 let 声明就会抛出错误。
但如果在嵌套的作用域内使用 let 声明一个同名的新变量，不会报错

```
var count = 30
if(condition) {
  let count = 40 // 不会抛出错误
}
```

- 循环中的绑定块级作用域的妙用

```
for(let i = 0; i < 10; i++) {
  //....
}
console.log(i) // ReferenceError: i is not defined
```

上面代码中，计算器只在 for 循环体中有效，在循环体外引用就会报错

```
var arr = [];
for(let i = 0; i < 10; i++) {
  arr[i] = function (){
    console.log(i)
  }
}
arr[6]() // 6
```

上面代码中，变量 i 是 let 声明的，当前的 i 只在本轮循环有效，所以每一次循环的 i 其实都是一个新的变量，所以最后输出的是 6。你可能会问，如果每一轮循环的变量 i
都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量 i 时，
就在上一轮循环的基础上进行计算。  
另外，for 循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域

```
for(let i = 0; i < 3; ++) {
  let i = 'abc'
  console.log(i)
}
// abc
// abc
// abc
// 输出了三次abc，表示函数内部的变量i与循环变量i不在同一个作用域，有各自单独的作用域
```

## 作用域链

###### 什么是自由变量

在如下代码中，console.log(a)要得到 a 变量，但是在当前作用域下没有定义 a（可对比下变量 b）。当前作用域没有定义的变量，这成为自由变量，自由变量的值如何得到---
向父级作用域寻找

```
var a = 100;
function fn(){
  var b = 200;
  console.log(a) // 这里的a在这里就是自由变量
  console.log(b)
}
```

###### 什么是作用域链

如果父级作用域也没有呢？，再一级一级向上查找，直到找到全局作用域还没有找到，就宣布放弃。这种一层一层的关系，就是作用域链

```
var a = 100
function f1(){
  var b = 200
  function f2(){
    var c = 300
    console.lg(a) // 自由变量，顺作用域链向父作用域链查找
    console.lg(b) // 自由变量，顺作用域链向父作用域链查找
    console.lg(c)
  }
  f2()
}
f1()
```

###### 关于自由变量的取值

关于自由变量的值，上面提到要到父级作用域中取，这种解释也会产生歧义

```
var x = 10
function fn(){
  console.log(x)
}
function show(f){
  var x = 20
  (function (){
    fn() // 10
  })()
}
show(fn)
```

在 fn 函数中，取自由变量 x 的值时，要到那个作用域取？—————要到创建 fn 函数的那个作用域中取，无论 fn 函数在哪里调用  
所以不要在用以上的说法，相比而言，用这句话更加合适，**自由变量的取值要到函数创建的那个作用域取值，这里强调是的'创建'而不是'调用'** ，这就是静态作用域
