#### 变量提升
在var时代，有一个特别的现象：不管变量声明是写在程序的那个角落，最后都会提到作用域的顶端去  
例如：  
```
var num
console.log(num) // undefined
num = 1

function getNum (){
  console.log(num) // undefined
  var num = 1
}
```

#### 变量提升的原理
<!-- JS是一变编译一遍执行，简单来说，所以的JS代码片段在执行之前都会被编译，只是这个编译的过程非常短暂(可能就只有几微妙、或者更短的时间)，紧接着这段代码就会被执行   -->
JS会经历编译和执行阶段，正是在这个短暂的编译阶段里，JS引擎会搜集所有的变量声明，并且提前让声明生效。至于剩下的语句，则需要等到执行阶段、等到执行到具体的某一句的时候才会生效。这就是变量提升背后的机制

#### 被禁用的变量提升
在let和const区别于var的一个重要特性——它们不存在变量提升

```
console.log(num) // Uncaught SyntaxError: Identifier 'num' has already been declared
let num = 1
```

如果改成const声明，也是一样的结局 ———— let和const声明的变量，它们的声明生效时机和具体代码的执行时机保持一致。  

#### let与const

> let关键字与var关键字

```
{
  var me = 'xiaozhang'
  cosnole.log(me) // 'xiaozhang'
}

console.log(me) // 'xiaozhang'

{
  let me1 = 'xiaoling'
  console.log(me1) // 'xiaoling'
}

console.log(me1) // '报错'
```

let和var非常相似，let区别与var的最关键的地方在于：当我们用let声明变量时，变量会被绑定到作用域上，而var是不感知作用域的  
而我们用let声明变量时，变量被绑定到了它被声明的那个代码块里。这时块作用域生效了，它表现出了和函数作用域相似的特征————出了作用域，就访问不到里面的变量

> cosnt 关键字

const关键字和let具备相同的生命周期特性————用const声明的变量，也会绑定到块作用域上

```
{
  const me = 'xiaozhang'
  console.log(me)
}

console.log(me) // 报错

const a; // 报错
```

const与let、var之间的区别：const声明的变量，必须在声明同时被初始化，否则会报错  
const声明的变量，在赋值过后，值不可以再被更改，否则同样会报错，如果是引用类型的属性值可以被修改，只要不修改应用的指向

#### 暂时性死区
暂时性死区本质：当我们进入当前作用域时，let或const声明的变量已经存在了，它们只是不允许被获取而已。想要获取它们，必须得等到代码执行到声明处

```
var me = 'xiaozhang'

{
  me = 'bear'
  let me
}

// Uncaught ReferenceError: Cannot access 'me' before initialization
```

在上面代码运行会直接报错，这是因为在ES6中有明确的规定：如果区块中有let或const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。假如我们尝试在声明前去使用这类声明，就会报错  
这一段会报错的危险区域，有一个专属的名字，叫'暂时性死区'。在上面代码中，以红线为界，上面的区域就是暂时性死区：

![图片](./../image/variablePromotions.png)





