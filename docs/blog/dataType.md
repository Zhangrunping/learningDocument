<style>
  .bgColor {
    border-radius：2px;
    background:#fff5f5;
    color:#ff502c;
    padding: .065em .4em
  }
</style>
<!-- <span class='bgColor'></span> -->
#### 数据类型
- 基本数据类型：String、Number、Boolean、Null、Undefined、Symbol、BigInt
- 引用数据类型：Object

> 基本数据类型和应用类型的区别
- 1、<span class='bgColor'>内存分配不同</span> 原始值是存储在栈(stack)中的简单数据段；引用值是存储在堆(heap)中的对象，也就是说存储在变量处的值是一个指针，指向存储对象的内存地址。
- 2、<span class='bgColor'>访问机制不同</span> 原始值可以直接访问到，引用值需要通过存储在栈空间中的引用地址去获取
- 3、<span class='bgColor'>赋值变量不同</span> 原始值赋值到新变量时会在栈内存中新开辟一个空间来存储新值，新旧两值互不影响；原始值赋值时实际上赋予的是该对象的引用指针，新旧对象共用同一个引用指针，属性方法共用。
- 4、<span class='bgColor'>参数传递不同</span> 原始值只是把变量的值传递给参数，之后参数和这个变量互不影响；引用值传递给参数的值是这个对象变量指向堆内存中数据实体的内存地址。函数内部修改参数，外部的对象也会受影响。

![图片](./../image/stack.jpg)

![图片](./../image/heap.jpg)


#### 判断数据类型方法
> typeof操作符---返回一个字符串，表示未经计算的操作数的类型
- 基础类型除null以外，返回正常结果
- 引用类型除function一危，一律返回object类型
- null，返回object类型，function返回function类型

```
  typeof null         // object
  typeof {}           // object
  typeof []           // object
  typeof ''           // string
  typeof true         // boolean
  typeof 1            // number
  typeof Symbol()     // symbol
```

> instanceof运算符---用于检测构造函数prototype属性是否出现在某个实例对象的原型上

```
  function Car (){}
  let car = new Car()
  car instanceof Car     // true
  car instanceof Object  // true
  car instanceof Array   // false
```

> Object.prototype.toString--- 通常需要检测所有类型，调用该方法封装

```
  function detectionType (data){
    return Object.prototype.toString.call(data).slice(8, -1).toLocaleLowerCase();
  }

  detectionType(Symbol())   // symbol
  detectionType([])         // array
  detectionType({})         // object
  detectionType('')         // string
  detectionType(true)       // boolean
```


#### 数据类型转换 
> 强制转换为<span class='bgColor'>String</span>  

1、toString方法，调用此方法不会影响原变量，它会将转换的结果方法，注意:null和undefined两个值没有toString方法，调用会报错  
2、String方法，对于Number和Boolean实际上是调用了toString()方法，null和undefined不会调用此方法，则直接转换成'null'和'undefined'
3、toString和String方法转换的目标是对象或者数组，对象则返回一个类型字符串，数组则返回该数组的字符串形式

```
  let n = 1
  let e = null
  let o = undefined
  let u = {name: 'xx'}
  let y = [1,2,3]
  
  n.toString()      // '1'
  e.toString()      // 'Uncaught TypeError: Cannot read properties of null (reading 'toString')'
  o.toString()      // 'Uncaught TypeError: Cannot read properties of null (reading 'toString')'
  String(n)         // '1'
  String(e)         // 'null'
  String(o)         // 'undefined'
  u.toString()      // [object Object]
  y.toString()      // 1,2,3
  String(u)         // [object Object]
  String(y))        // 1,2,3
```

> 强制转换为<span class='bgColor'>Number</span>

方式一：Number()函数  
1、字符串转数字，纯数字的字符串，则直接转换为数字；字符串中含有有非数字的内容，则转换为NaN；字符串是一个空串或者全是空格的字符串，则转换为0  
2、布尔值转数字，true返回值1，false返回值0  
3、undefined转换数字，返回值NaN  
4、null转换数字，返回值0  
5、Number方法接收数值作为参数，既能识别负的十六进制，也能识别0开头的八进制，返回值永远是10进制  

```
  Number('1')         // 1
  Number('1n')        // NaN
  Number('')          // 0
  Number(true)        // 1
  Number(null)        // 0
  Number(undefined)   // NaN
  Number(023);        // 19
  Number(0x12);       // 18
```

方式二：parseInt() parseFloat()  
parseInt方法将一个字符串转换为一个整数，可以将一个字符串中的有效整数内容取出来，然后转换为Number  
parseFloat() 函数解析一个参数（必要时先转换为字符串）并返回一个浮点数  

```
  parseInt('.21')          // NaN
  parseInt("10.3")         // 10
  parseFloat('.21')        // 0.21
  parseFloat('.d1')        // NaN
  parseFloat("10.11.33")   // 10.11
  parseFloat("4.3years")   // 4.3
  parseFloat("He40.3")     // NaN
```

> 强制转换为<span class='bgColor'>Boolean</span>  

除''、null、undefined、+0、-0、和NaN转布尔类型为false，其余的都是true

```
  Boolean('')               // false
  Boolean(undefined)        // false
  Boolean(null)             // false
  Boolean(0)                // false
  Boolean(NaN)              // false
  Boolean({})               // true
  Boolean([])               // true
  Boolean(new Boolean(false))  // true
```

> 自动转换为<span class='bgColor'>String</span>
 
规则：字符串的自动转换，主要发生在字符串的加法运算时，当一个值是字符串，另一个值为非字符串，则后者转为字符串

```
  '1' + 1          // '11'
  '1' + false      // '1false'
  '1' + true       // '1true'
  '1' + {}         // '1[object Object]'
  '1' + []         // '1'
  '1' + [1, 2, 3]  // '11,2,3'
  '1' + undefined  // '1undefined'
  '1' + null       // '1null'
  '1' + function(){}  // '1function (){}'
```

> 自动转换为<span class='bgColor'>Number</span>

规则：算术运算符（+ - * /）跟非Number类型的值进行运算时，会将这些值转换Number在进行运算。除了字符串的加法运算

```
  true + 1         // 2
  2 + null         // 2
  undefined + 1    // NaN
  NaN + 2          // NaN
  '2' * '2'        // 4
  false / 5        // 0
  'N' - 1          // NaN
```


> 自动转换为<span class='bgColor'>Boolean</span>  

规则：遇到预期为布尔值的地方(例如if语句的条件部分)，会将非布尔值的参数自动转换为布尔值，系统内部自动调用Boolean函数

```
  if('hello') {
    console.log('world');
  }
```


 




