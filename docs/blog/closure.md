<style>
  .bgColor {
    border-radius：2px;
    background:#fff5f5;
    color:#ff502c;
    padding: .065em .4em
  }
</style>
<!-- <span class='bgColor'></span> -->
#### 如何理解JavaScript中的闭包
<span class='bgColor'>闭包</span>：有权访问另一个函数作用域中的变量的函数  
<span class='bgColor'>特性</span>：函数嵌套函数，内部函数可以引用外部函数的变量和参数；参数和变量不会被垃圾回收机制所回收  
<span class='bgColor'>优点</span>：变量长期存储在内存中；避免全局变量的污染；私有成员的存在   
<span class='bgColor'>缺点</span>：常驻内存，会加大内存的使用量，可能会造成内存泄漏

#### 代码理解闭包

```
  for (var i = 0; i < 5; i++) {
    setTimeout(function() {
        console.log(i);
    }, 1000);
  }

  console.log(i);
  // 输出 5 5 5 5 5 5
```
上面代码执行过程：在函数执行时，也就是1000ms后。它向上层作用域(全局作用域)去找变量i。此时for循环早已经执行完成，i的最终状态为5。所以1000ms后，当这个函数第一次真正被执行的时候，引用到的i值已经是5了。函数执行时整体的作用域状态示意如下

![图片](./../image/slosure1.png)

对应的作用域链关系如下：

![图片](./../image/slosure2.png)

接下来持续4次，每个1000ms，都会有一个一模一样的setTimeout回调被执行，输出的也是同一个全局i，所以每一次的输出都是5

#### 改变它的三种思路

> 第一种思路-使用setTimeout函数的第三个参数传入i

```
  for (var i = 0; i < 5; i++) {
    setTimeout(function(j) {
        console.log(j);
    }, 1000, i);
  }
```

> 第二种思路-在setTimeout上套一层函数，利用这个外部函数的入参来缓存每个循环中的i值

```
  var output = function (i){
    setTimeout(() => {
      console.log(i);
    }, 1000);
  }

  for (var i = 0; i < 5; i++) {
    output(i)
  }
```

第三种思路-在setTimeout上套一层立即执行函数

```
  for (var i = 0; i < 5; i++) {
    (function (j){
      setTimeout(() => {
      console.log(j);
      }, 1000);
    })(i)
  }
```

#### 闭包-循环体

```
  function test (){
    var num = []
    var i

    for (i = 0; i < 10; i++) {
        num[i] = function () {
            console.log(i)
        }
    }

    return num[9]
  }

  test()() // 输入 10

  var test = (function() {
    var num = 0
    return () => {
        return num++
    }
  }())

  for (var i = 0; i < 10; i++) {
    test()
  }

  console.log(test()) // 输出 10
```

#### 闭包-复杂作用域
<span class='bgColor'>解题思路步骤</span>
    
> step1: 读题

```
  var a = 1

  function test(){
    a = 2
    return function (){
      console.log(a);
    }
    var a = 3
  }

  test()()
```

> step2: 画图

  - 分层：从内向外画，最内层，是一个匿名函数。匿名函数的外层，是test函数的作用域，在外层，就是全局作用域了
  
  ![图片](./../image/slosure3.png)

  - 找变量：全局变量里的有a =1，最内层匿名函数里的a当前作用域不存在，主要需要分析test函数的a  
    1、<span class='bgColor'>变量提升</span>：test函数中，a =2在先，var a =3在后。但是函数作用域内部也是存在变量提升的，这个var会被提升到函数的顶部，所以test函数体等价于下面这种情况

    ```
      function test(){
        // 声明var被提前
        var a = 2
        return function (){
          console.log(a);
        }
        a = 3
      }
    ```

    2、<span class='bgColor'>作用域规则</span>：结合setp1里划分的层级和上一步分析出来test函数中的变量情况，可以看出匿名函数实际拿到的是a就是test函数作用域下的a。那么这个a是2还是3呢?  
    这里我们匿名函数被定义的时候a =3赋值动作还没有发生，因此它拿到的是2  
    作用域的划分，是在**书写的过程中，根据你把它写在哪个位置来决定的**。想这样划分出来的作用域，遵循的就是词法作用域模型  

    现在将分析出来的变量一个一个填进对应的地盘：  

    ![图片](./../image/slosure4.png)

    作用域链如下

    ![图片](./../image/slosure5.png)

#### 闭包的应用

> 模拟私有变量

```
  const User =  (function (){
    // 定义私有变量
    let _passWord;

    class User {
      constructor(userName, passWord){
        _passWord = passWord
        this.userName = userName
      }

      login(){
        console.log(this.userName, _passWord)
      }
    }

    return User;
  })()
```

> 柯里化

柯里化是指将多个入参的函数，转化为需要更少入参的函数的方法  
柯里化是把接受 n 个参数的 1 个函数改造为只接受 1个参数的 n 个互相嵌套的函数的过程。也就是fn(a, b, c)会变成fn(a)(b)(c)

```
  function generateName (prefix){
    return function (type){
      return function (itemName){
          return `${prefix}-${type}-${itemName}`
      }
    }
  }


  let getPrefix = generateName('丁丁网')
  let getType = getPrefix('书籍')
  let itemName1 = getType('JavaScript高级程序设计')
  let itemName2 = getType('你不知道的JavaScript')
  console.log(itemName1);               // 丁丁网-书籍-JavaScript高级程序设计
  console.log(itemName2);               // 丁丁网-书籍-你不知道的JavaScript
  console.log(generateName('大麦网')('生活用品')('毛巾'));           // 大麦网-生活用品-毛巾
```


> 偏函数

偏函数是指固定函数的某一个或几个参数，然后返回一个新的函数(这个函数用于接收剩下的参数)  
比如：有一个函数有10个入参，可以只固定2个参数入参，然后返回一个需要8个入参的函数

```
  function generateName(prefix){
    return function (type, itemName){
      return `${prefix}-${type}-${itemName}`
    }
  }

  let getPrefix = generateName('丁丁网')

  console.log(getPrefix('书籍', 'JavaScript高级程序设计'));
  console.log(generateName('大麦网')('生活用品', '毛巾'));
```

#### 垃圾回收机制
每隔一段时间，js的垃圾收集器就会对变量做'巡检'，当它判断一个变量不在被需要之后，它就会把这个变量所占用的内存空间给释放掉。这个过程叫做垃圾回收

- 引用计数法
  在引用计数法的机制下，内存中的每个值都会对应一个引用计数。当垃圾收集器感知到某个值的引用计数为0，就判断它'没用了'，随即这块内存就会被释放  
  缺点：引用计数法无法甄别循环引用场景下的'垃圾'

- 标记清除法
  在标记清除算法中，一个变量是否被需要的判断标准，是它是否抵达
  标记阶段：垃圾收集器会先找到根对象，在浏览器里是window，在node里是global。从根对象出发，垃圾收集器会扫描所有可以通过根对象触及的变量，这些对象会被标记为'可抵达'  
  清除阶段：没有被标记为'可抵达'的变量，就会被认为是不需要的变量，这波变量就会被清除
#### 闭包与内存泄漏

> 内存泄漏

该释放的变量(内存垃圾)没有被释放，仍然霸占着原有的内存不松手，导致内存占用不断攀高，带来性能恶化，系统崩溃等一列问题，这种现象就叫内存泄漏

> 闭包是如何导致内存泄漏的

```
  var theThing = null;
  var replaceThing = function () {
    var originalThing = theThing;
    var unused = function () {
      if (originalThing) // 'originalThing'的引用
      console.log("嘿嘿嘿");
      };
    theThing = {
      longStr: new Array(1000000).join('*'),
      someMethod: function () {
        console.log("哈哈哈");
      }
    };
  };
setInterval(replaceThing, 1000);
```

这段代码里， unused 是一个不会被使用的闭包，但和它共享同一个父级作用域的 someMethod，则是一个 “可抵达”（也就意味着可以被使用）的闭包。unused 引用了 originalThing，这导致和它共享作用域的 someMethod 也间接地引用了 originalThing。结果就是 someMethod “被迫” 产生了对 originalThing 的持续引用，originalThing 虽然没有任何意义和作用，却永远不会被回收。不仅如此，originalThing 每次 setInterval 都会改变一次指向（指向最近一次的 theThing 赋值结果），这导致无法被回收的无用 originalThing 越堆积越多，最终导致严重的内存泄漏。

> 内存泄漏成因分析
- 全局变量  
  ```
    function test(){
      name = 'zzp'
   }
  ```  

- 忘记清除setInterval和setTimeout
  ```
    setTimeout(function (){
      // ....
    },1000)

    setInterval(function (){
      // ....
    });
  ```

> 清除不当的DOM
  ```
    const myDiv = document.getElementById('myDiv')

    function handleMyDiv() {
      // 一些与myDiv相关的逻辑
    }

    // 使用myDiv
    handleMyDiv()

    // 尝试”删除“ myDiv
    document.body.removeChild(document.getElementById('myDiv'));
  ```





