+++ 
draft = true
date = 2020-01-23
title = "JS基础汇总"
description = ""
slug = ""
authors = []
tags = ["js", "八股文"]
categories = ["js"]
externalLink = ""
series = []
+++

#### 0.1 + 0.2 为什么不等于 0.3
- [为什么0.1+0.2不等于0.3](https://segmentfault.com/a/1190000012175422)
- [非科班前端人的一道送命题：0.1+0.2 等于 0.3 吗？](https://zhuanlan.zhihu.com/p/363133848)

现在浏览器多数采用双精度即64存储浮点数
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/987ba86bba9341eabd3e70bcc5237ddd~tplv-k3u1fbpfcp-zoom-1.image">
- 第1位： 符号位，正数0，负数1
- 2-11位： 指数位阶数+偏移量，阶数是：2^e-1 - 1
- 12-64: 小数位， 即二进制小数点后面的数

> 0.1+0.2 不等于 0.3 ，因为在 0.1+0.2 的计算过程中发生了两次精度丢失。第一次是在 0.1 和 0.2 转成双精度二进制浮点数时，由于二进制浮点数的小数位只能存储52位，导致小数点后第53位的数要进行为1则进1为0则舍去的操作，从而造成一次精度丢失。第二次在 0.1 和 0.2 转成二进制浮点数后，二进制浮点数相加的过程中，小数位相加导致小数位多出了一位，又要让第53位的数进行为1则进1为0则舍去的操作，又造成一次精度丢失。最终导致 0.1+0.2 不等于0.3 。

解决
- toFixed var c = 0.1 + 0.2; c.toFixed(2) === "0.30"， 在做Number转换

<!--more-->

#### typeof null 为什么等于object
简单来说，就是js最初遗留的bug，数据在底层采用二进制，js设计之初才用32位字节储存，由标志位(1-3位)和数值组成。同时认为 前三位是 000 判定为object类型
- 对象 000
- null 全为 000
- 整数 1
- 浮点数 010
- 字符串 100
- 布尔 110
- undefined 全为1

#### 描述一下 reduce 的执行

array.reduce(callback(accumulator, currentVal, index, array), initiaValue)
reduce 函数接受 4 个参数

- accumulator 当前累加器的返回值
- currentVal 数组正在处理的元素
- index 当前正在处理的索引
- array 当前操作数组
- initiaValue 有值，那么 callback 函数的 accumulator 为 initiaVale, currentVal 为数组的第二项值，如果没有提供改值，accumulator 为数组 array[0], currentVal 为数组 array[1]
- **注意:** 空数组 && initiaVal 不传，则会报错;

```
    [].reduce(callback)  // Reduce of empty array with no initial value
    [].reduce(callback, null)  // null
    [].reduce(callback, '')    // '',
    [].reduce(callback, undefined) // undefined
```

##### reduce 累加求和

```
    const arr = [];
    const sum = arr.reduce((prevVal, curVal, index, arr) => {
        console.log(`prev:${prevVal}`);
        console.log(`cur: ${curVal}`);
        return prevVal + curVal;
    }, 0);
```

##### reduce 打平二位数组成一维

```
    const arr = [[0, 1], 2, 3], [4, 5]];
    const sum = arr.reduce((prevVal, curVal, index, arr) => prevVal.concat(curVal));
```

#### 扁平化数组
concat 不会改变原数组，返回新增后的数组   
shift 删除数组第一个元素，并且返回数组被删除元素，改变原数组
unshift 向头部添加一个元素，并且返回数组最新长度，改变原数组
> 栈方式
```javascript
  function deelFlat(arrs) {
    let result = [];
    while (arr.length) {
      const value = arr.shift();
      if (Array.isArray(value)) {
        result = result.concat(deepFlat(value));
      } else {
        result.push(value);
      }
    }

    return result;
  }


  // 方式二，向数组头部增加值
  function flatArray(arr) {
    const result = [];

    while (!!arr.length) {
      let item = arr.shift();
      if (Array.isArray(item)) {
        arr.unshift(...item);
      } else {
        result.push(item);
      }
    }

    return result;
  }
```

> reduce
```javascript
    function deelFlat(arrs) {
      return arrs.reduce((prev, cur) => {
        return [].concat(prev, Array.isArray(cur) ? deelFlat(cur) : cur);
      }, []);
    }
```

>toString
```javascript
  // before [1,2,3,[1, [2]], [1, [2, [3]]]]
  // after [1, 2, 3, 1, 2, 3, 4, 2, 3, 4]

  const arr = [1,2,3,[1, [2]], [1, [2, [3]]]]
  arr.toString().split(',')
```

> Array.prototype.flat(Infinity)  // Infinity无限大
```javascript
  const arr = [1,2,3,[1, [2]], [1, [2, [3]]]]
  arr.flat(Infinity) // [1, 2, 3, 1, 2, 3, 4, 2, 3, 4]
```

##### 计算数组中每个元素出现的次数

```
// method1

const arr = ['Alice', 'Bob', 'Tiff', 'Bruce', 'Alice'];
const sum = arr.reduce((prevVal, curVal, index, arr) => {
  if (prevVal[curVal]) {
    prevVal[curVal] = prevVal[curVal] + 1;
  } else {
    prevVal[curVal] = 1;
  }

  return prevVal;
}, {});

```

```
// method2

const arr = ['Alice', 'Bob', 'Tiff', 'Bruce', 'Alice'];
const sum = arr.reduce((obj, curVal, index, arr) => {
  if (curVal in obj) {
    obj[curVal]++;
  } else {
    obj[curVal] = 1;
  }

  return obj;
}, {});

```

##### 数组去重

```
const arr = [1, 2, 1, 2, 3, 5, 4, 5, 3, 4, 4, 4, 4];

const sum = arr.reduce((prevVal, curVal, index, arr) => {
  if (!prevVal.includes(curVal)) {
    prevVal.push(curVal);
  }
  return prevVal;
}, []);

```

#### 判断数组的几种方式
- obj.__proto__ === Array.prototype
- obj instanceof Arrray
- ArraY.isArray(obj)
- Object.prototype.toString.call(obj).slice(8, -1) // 默认 "[object String]"
- Array.prototype.isPrototypeOf(obj)

#### 什么是伪数组，伪数组有哪些，伪数组转化成数组有哪些方法

> 描述

- 伪数组是一个**对象**
- 伪数组具有 length 属性
- 不具备数组的方法，但是按照索引的方式存储数据

> 常见的伪数组： arguments, Dom 对象列表等

> 伪数组转化数组的几种方式

- Array.from(xxx)
- Array.prototype.slice.call(xxx);

#### 扩展运算符、rest

- rest 参数只包含哪些没有对应形参的实参, reet 是一个伪数组
- arguments 包含了传给函数的所有实参, arguments 是一个伪数组

**rest 注意点**

> 1.函数的 length 属性不包括 rest 参数

```javascript
function fn(a, b) {} //  fn.length ===  2
function fn2(a, ...rest) {} // fn.length === 1
```

> 2.rest 参数必须是函数形参最后一位

```javascript
function fn(a, b, ...rest) {} // success
function fn2(a,...rest, b) {} // Uncaught SyntaxError: Rest parameter must be last formal parameter
```

**扩展运算符**

> 1.拓展运算符可以看做是 rest 参数的逆运算  
> 2.作用于普通函数调用，如：array.push(...spread);，也可用于合并数组、对象, 将字符串转换为数组 [...'abcd'] === [a,b,c,d]

#### call/apply/bind 相同点以及不同点

**相同点**

- 改变函数执行上下文，通俗一点就是改变运行函数时 this 的指向
- 接受第一个值代表函数后续的执行上下文， 传入 null 或者 undefinedn 那么函数上下文将指向 window

**不同点**

- call(context, a1, a2, ...) 接受参数为参数序列
- apply(context, [a1, a2, ...]) 接受参数为数组
- const newFn = fn.bind(context); newFn(a1, a2) 先执行 bind 函数改变函数上下文，返回一个函数，后续再执行传入相关参数

```javascript
const obj = {
  name: 'levi',
  showName: function name(params) {
    console.log(this.name);
  },
  output: function name(age, area) {
    console.log(age, area);
  }
};

const student = {
  name: 'zz'
};

obj.showName(); // levi
obj.showName.call(student); // zz
obj.showName.call(null); //  this指向了window this.name === undefined
obj.output.call(student, 20, 'shanghai'); // 20 shanghai
obj.output.apply(student, [20, 'shanghai']); // 20 shanghai

const fn = obj.showName.bind(student);
const fn2 = obj.output.bind(student);
fn(); // zz
fn2(20, 'bj'); // 20 bj
```

> 应用

```javascript
Math.max(null, [1, 2, 3, 4]); // 4
[].slice.call([1, 2, 3, 4], 2); // 3 4

var a = [1];
a = Array.prototype.concat.apply(a, [2, 3, 4]);
console.log(a); // 1,2,3,4
```
##### 实现call
```javascript
    /** 手动实现call */
  Object.prototype.LeviCall = function (context) {
    const params = [];
    const new_context = context || window;
    new_context.fn = this;

    for (let i = 1; i < arguments.length; i++) {
      params.push(arguments[i]);
    }

    new_context.fn(...params);
    delete new_context.fn;
  };
```

##### 实现apply
```javascript
  /** 手动实现apply */
  Object.prototype.LeviApply = function (context) {
    const params = arguments[1] || [];
    const new_context = context || window;
    new_context.fn = this;

    new_context.fn(...params);
    delete new_context.fn;
  };
```


##### 实现bind
```javascript
    function show(name, age, area) {
      this.tag = '我是 show 方法的tag';
      this.name = name;
      console.log(this.name, age, area);
    }
    show.prototype.num = 20;

    /** 手动实现bind */ 
    Object.prototype.LeviBind = function () {
      if (typeof this !== 'function') {
        throw new Error('this is not function');
      }

      const _this = this;
      const mediaFn = function () {};
      const transfer = Array.prototype.slice.call(arguments, 0, 1);
      const restParams = Array.prototype.slice.call(arguments, 1);

      const fn = function () {
        // 实例化对象从原型链找construtor，最终会找到show.prototype
        _this.apply(this.constructor === _this ? this : transfer || window, [
          ...restParams,
          ...arguments
        ]);
      };

      mediaFn.prototype = _this.prototype; 
      fn.prototype = new mediaFn(); // 引入中介函数，防止实例对象修改 show原型对象的属性及方法
      return fn;
    };

    const newFn = show.LeviBind(obj);
    const fn = new newFn(1, 2, 3); // 如果指向了new 操作，开始传入obj不起作用，里面this将指向原始调用方法(show)
    fn.__proto__.num = 21;
    console.log(fn.num, show.prototype.num); // 21 20  实例化后对象不会修改原来 构造函数show原型对象的属性及方法
```

#### js 数据类型

- 基本数据类型

  - String
  - Number
    - JavaScript 中只有一种数字类型：基于 IEEE 754 标准的双精度 64 位二进制格式的值（-(253 -1) 到 253 -1）,超出这 2 值就不是安全的了,会返回 Infinity
    - 要检查值是否大于或者小于临界值，可以使用 Number.MAX_VALUE 和 Number.MIN_VALUE 判断
  - BigInt

    - BigInt 类型是 JavaScript 中的一个基础的数值类型，可以用任意精度表示整数。使用 BigInt，您可以安全地存储和操作大整数，甚至可以超过数字的安全整数限制。BigInt 是通过在整数末尾附加 n 或调用构造函数来创建的

      > BigInt 操作方法
      > BigInt('9007199254740995') => 9007199254740995n  
      > var a = 9007199254740995n

      ```javascript
      //  BigInt 不能与 number 类型直接比较，会直接返回 false, 不允许BigInt 和 Number混合隐式类型转化，会报错
      3n === 3; // false
      3n == 3; // true
      typeof 3n; // bigint
      typeof 3; // number

      3n + 3; // Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions
      ```

  - Boolean
  - undefined
  - Symbol
    > es6 新加入的类型吗 Symbol 不可能更改，可以用作 Object 的 key </br>
    > Symbol(1) === Symbol(1) // false

- 引用数据类型

  - Object
  - Null  
     **javascript 历史遗留 bug，诞生就是如此**
    > 1. 从逻辑角度来看，null 值表示一个空对象指针，而这正是使用 typeof 操作符检测 null 值时会返回“object”的原因。
    > 2. 在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于 null 代表的是空指针（大多数平台下值为 0x00），因此，null 的类型标签是 0，typeof null 也因此返回 "object"
    > 3. 在 Javascript 中二进制前三位都为 0 的话会被判断为 Object 类型，null 的二进制表示全为 0，自然前三位也是 0，所以执行 typeof 时会返回"object"。
    - Object.prototype.toString.call(null) // "[object Null]"
    - typeof null === 'Object' // true

#### 数据类型检测方式

- typeof

  > 弊端，不能区分引用数据类型，另外 typeof null === object

  ```javascript
  typeof 1; // number
  typeof 'a'; // string
  typeof Symbol('b'); // symbol
  typeof 3n; // bigint
  typeof false; // boolean
  typeof undefined; // undefined
  typeof null; // object
  typeof []; // object
  typeof fn; // function
  typeof {}; // object
  ```

- instanceof
  instanceof 不太准确，原型链容易收到更改

```javascript
  var a = [];
  a.__proto__ = null
  a instanceof Array // false
  [] intsanceof Array // true
  [] instanceof Object // true
```

- constructor
  constructor 不太准确，原型链容易收到更改

```javascript
var c = []
c.__proto__.constructor  ==== Array // true

c.__proto__.constructor = null
c.__proto__.constructor  ==== Array // false

```

- Object.prototype.toString.call(xxx) **推荐**

```javascript
  Object.prototype.toString.call(1) // "[object Number]"
  Object.prototype.toString.call('a') // "[object String]"
  ...
  Object.prototype.toString.call(undefined) // "[object Undefined]"
  Object.prototype.toString.call([]) // "[object Null]"
  Object.prototype.toString.call(null)  // "[object Null]"
  Object.prototype.toString.call(3n) // "[object BigInt]"
```

#### 类型转换(隐式 && 显式)

- 转换的类型只有三种：to Number,to String,to Boolean
- 当基本类型转换成上述类型时会调用：Number() ,String(), Boolean()
- 当 0/undefined/null/false/NaN 转换为 boolean 为 false, 其他都为 true
- (所有对象转换 boolean 都为 true)
- 引用类型与基本类型做比较或者基本运算的时候，先把引用类型转换成基本类型，调用 valueOf()/toString() 方法

##### 作比较
- 字符串与number做比较， string转换number在比较
- string与boolean比较， string转换boolean在比较
- boolean与nunber比较，boolean转number在比较
- 对象与字符串比较，对象调用toString在比较
- 对象与number比较，对象先转化为字符串，在转化为数字，在比较
- 对象与boolean比较， 对象先转化为字符串，在转化数字，在转化为布尔值，做比较

##### 做运算
- string + number, number会转化string类型相加
- string + boolean, 直接相加 "2" + false = "2false"
- number + boolean, boolean转为number，在做类型
- 对象的话，先调用对象valueOf => 在调用toString(); 如果返回基本类型就停止，按照上述规则来运算
- **2个引用数据类型比较， 直接返回false，堆内存地址不同**

**引用类型**
> 首先调用 valueOf，如果执行结果是原始值，返回
> valueOf 返回的不是原始值，则调用 toString,如果执行结果是原始值,返回，如果不是，报错
> 当使用显示类型转换成 String 时，执行顺序则是先调用 toString,其次调用 valueOf

({})+[] // [object object]  
{} + [] // 0  
[] + {} // [object object]
在 js 中{}代表复合语句，在一些 js 解释器会将开头的  {}  看作一个代码块，而不是一个 Object（在 es6 以前只有函数作用域与全局作用域，还没有块级作用域）而这里的{}只是空符号，不表明任何意思。这里的+[]是一个隐式转换，所以参与运算的只有+[]，在这里将[]转换成了 number 类型，所以得出结果为 0

```javascript
let obj = {
  value: '你好啊',
  num: 2,
  toString: function () {
    return this.value;
  },
  valueOf: function () {
    return this.num;
  }
};
console.log(obj + '明天'); // '2明天'

1 + '1'; // 11
true + 0; // 1
{
}
+[]; // 0
({} + []); // [object object]
4 + {}; // 4 [object object]
4 + [1]; // 41
'a' + +'b'; // aNaN
console.log([] == 0); // true
console.log(![] == 0); // true 有取反符号在前面，先执行取反操作(转化成boolean)，记住，对象的boolane为true，取反的为false
console.log([] == ![]); // true 
console.log([] == []); // false  引用类型之间比较的话， 因为引用存放的是指针地址，直接比较的话，其实比较的是指针，所以报错
console.log({} == !{}); // false
console.log({} == {}); // false
```

#### js 遍历方式

> forEach,map,filter,every,some，reduce 等.
> forEarch 返回值是 undefined 没有意义，map 返回具体值，可以改变用返回值

#### 原型 && 原型对象 && 原型链

概念分为构造函数、实例对象、原型对象、原型链

<br />

下面谈谈我的理解  
javascript 万物皆对象，每个实例都有一个**proto**属性，指向了他构造函数的原型对象
每个构造函数都有一个 prototype 属性，指向了他的原型对象

> 实例对象创建之间判断指向关系有 **proto** 代表指向关系 指向了原型对象

```javascript
function A() {} // A 构造函数
var a = new A(); // a 实例对象

A.prototype; // 原型对象
a.__proto__ === A.prototype;
```

所有函数都是由 Function 创建

```javascript
A.__proto__ === Function.prototype; // true

Function.__proto__ === Function.prototype; // true
```

Object 刚讲的是顶级函数，所以也是函数：（所有的鱼都归猫管哈哈哈哈哈）

```javascript
Object.__proto__ === Function.prototype;
```

**所有的对象都是由 Object 构造函数创建的**, 所以对象**proto**指向了 Object 的构造函数：

```javascript
A.prototype.__proto__ === Object.prototype;
```

Object.prototype 也是对象，比较特殊，指向了 null

```javascript
Object.prototype.__proto__ === null;
```

原型链(一种访问机制)：

- 在访问对象的某个成员的时候会先在对象中找是否存在
- 如果当前对象中没有就在构造函数的原型对象中找
- 如果原型对象中没有找到就到原型对象的原型上找
- 直到 Object 的原型对象的原型是 null 为止

<div style="text-align: center">
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5075b34d796740a48eafce7439d7fd91~tplv-k3u1fbpfcp-zoom-1.image" style="width: 600px; margin: 0 auto; display: inline-block">
</div>

#### js 实现继承的方式

- ##### 原型链继承

  > 构造函数，原型，实例对象之间关系，每个构造函数中有属性(prototype)指向个原型对象(F.prototype)，原型对象上挂在公有属性、方法供子类分享， 同时原型对象有个 constructor 指向了构造函数。 实例对象有属性**proto**指向了原型对象， **继承的本质:** 重写构造函数的原型对象，使其重新指向另一个构造函数的实例

  ```javascript
  function Animal() {
    this.tag = '我是动物';
    this.area = ['china', 'japanese'];
  }

  Animal.prototype.desc = ['red', 'green'];
  Animal.prototype.say = function () {
    console.log(this.name);
  };

  function Dog(name) {
    this.name = name;
  }

  Dog.prototype = new Animal();

  const dog = new Dog('dogName');
  const dog2 = new Dog('dogName2');

  dog.area.push('uk');
  console.log(dog2.area); // dog.area = ['china', 'japanese', 'uk']  dog2.area = ['china', 'japanese', 'uk']
  ```

  **样例总结**

- 父构造函数内部属性在在执行后，会挂在相应子构造函数原型对象上， 修改各自原型对象上属性(包含引用)，其他子类不受影响，例如： area 属性

  **弊端**

- 无法实现多继承
- 创建子类实例时，无法向父类构造函数传参
- 多个实例对引用类型的操作会被篡改。

- ##### 构造函数继承

  > 使用父类构造函数来增加子类，等同于复制父类的属性及方法给子类
  > 核心代码是 Animal.call(this)，创建子类实例时调用 Animal 构造函数，于是 Dog,Cat 的每个实例都会将 Animal 中的属性复制一份。可以实现多继承（call 多个父类对象）

  ```javascript
  function Animal() {
    this.tag = '我是动物';
    this.area = ['china', 'japanese'];
  }

  Animal.prototype.desc = ['red', 'green'];
  Animal.prototype.say = function () {
    console.log(this.name);
  };

  function Dog(name) {
    Animal.call(this);

    this.name = name;
  }

  const dog = new Dog('dogName');
  dog.area.push('uk');
  dog.desc.push('blue'); // Cannot read property 'push' of undefined
  dog.say(); //  dog.say is not a function
  console.log(dog, dog.area, dog.desc); // dog.area = ['china', 'japanese', 'uk']  dog.desc = undefined
  ```

  **弊端**

  - 实例并不是父类的实例，只是子类的实例
  - 只能继承父类构造函数的属性及方法，无法继承父类原型对象属性及方法
  - 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

- ##### 组合继承

  > 实例对象 instance1 上的两个属性就屏蔽了其原型对象 SubType.prototype 的两个同名属性。所以，组合模式的缺点就是在使用子类创建实例对象时，其原型中会存在两份相同的属性/方法

  - 既是子类的实例，也是父类的实例
  - 不存在引用属性共享问题
  - 可传参
  - 可传参

  ```javascript
  function Animal() {
    this.tag = '我是动物';
    this.area = ['china', 'japanese'];
  }

  Animal.prototype.desc = ['red', 'green'];
  Animal.prototype.say = function () {
    console.log(this.name);
  };

  function Dog(name) {
    Animal.call(this);

    this.name = name;
  }

  Dog.prototype = new Animal();
  const d1 = new Dog('d1');
  const d2 = new Dog('d2');

  d1.area.push('uk');
  console.log(d1, d2); // d1.area =  ["china", "japanese", "uk"]  d2.area = ['china', 'japanese']
  ```

  **弊端**

  调用了两次父类构造函数，生成了两份实例（子类实例属性将子类原型上的那份屏蔽）

- ##### 原型式继承

  > Object(xxx) => 创建一个空对象后来，空对象原型指向了 a.**proto** === xxx 、 a instanceof xxx // error instancece 右侧接受构造函数
  > instanceof 运算符用于检测构造函数的 prototype 属性是否出现在某个实例对象的原型链上。
  > object instanceof constructor(构造函数)

  ```javascript
  function object(obj) {
    function F() {}
    F.prototype = obj;
    return new F();
  }

  const anotherPerson = object(person);
  anotherPerson.name = 'Greg';
  anotherPerson.friends.push('Rob');

  var yetAnotherPerson = object(person);
  yetAnotherPerson.name = 'Linda';
  yetAnotherPerson.friends.push('Barbie');

  alert(person.friends); //"Shelby,Court,Van,Rob,Barbie"
  ```

  **弊端**

  - 原型链继承多个实例的引用类型属性指向相同，存在篡改的可能
  - 无法传递参数

- ##### 寄生式继承
  核心：在原型式继承的基础上，增强对象，返回构造函数

```javascript
function createAnther(originObj) {
  var clone = Object.create(originObj);
  clone.fn = function () {
    console.log(this.name);
  };

  return clone;
}

var anotherPerson = createAnther(person);
anotherPerson.sayHi(); // xxx
```

**弊端(同原型式继承)**

- 1.原型链继承多个实例的引用类型属性指向相同，存在篡改的可能
- 2.无法传递参数

- ##### 寄生组合式继承

  > 这个例子的高效率体现在它只调用了一次 SuperType 构造函数，并且因此避免了在 SubType.prototype 上创建不必要的、多余的属性。于此同时，原型链还能保持不变；因此，还能够正常使用 instanceof 和 isPrototypeOf()

  ```javascript
  // 利用原型链继续往上调用
  function inheritPrototype(subType, superType) {
    var prototype = Object.create(superType.prototype); // 创建对象，创建父类原型的一个副本
    prototype.constructor = subType; // 增强对象，弥补因重写原型而失去的默认的constructor 属性
    subType.prototype = prototype; // 指定对象，将新创建的对象赋值给子类的原型
  }

  // 父类初始化实例属性和原型属性
  function SuperType(name) {
    this.name = name;
    this.colors = ['red', 'blue', 'green'];
  }

  SuperType.prototype.sayName = function () {
    alert(this.name);
  };

  // 借用构造函数传递增强子类实例属性（支持传参和避免篡改）
  function SubType(name, age) {
    SuperType.call(this, name);
    this.age = age;
  }

  // 将父类原型指向子类
  inheritPrototype(SubType, SuperType);

  // 或者
  Subtype.prototype = Object.create(Super.prototype);
  Subtype.prototype.constructor = Subtype;

  // 新增子类原型属性
  SubType.prototype.sayAge = function () {
    alert(this.age);
  };

  var instance1 = new SubType('xyc', 23);
  var instance2 = new SubType('lxy', 23);

  instance1.colors.push('2'); // ["red", "blue", "green", "2"]
  instance1.colors.push('3'); // ["red", "blue", "green", "3"]
  ```

- ##### ES6 类继承(extends)
  > 推荐

```javascript
    class Parent {
      constructor(name) {
        this.name = name;
      }

      sayName() {
        return this.name;
      }
    }

    class Child extends Parent {
      constructor(name, age) {
        super(name);
        this.age = age;
      }

      sayAge() {
        return this.age;
      }
    }

    const levi = new Child('levi', 20);
    const jack = new Child('jack', 21);
    console.log(levi, jack);
```

#### 作用域 && 作用域链
参考文章
- [JavaScript基础系列---执行环境与作用域链](https://segmentfault.com/a/1190000014899566)
- [JavaScript执行环境 + 变量对象 + 作用域链 + 闭包](https://www.cnblogs.com/no-particular/archive/2013/01/31/2887293.html)


```remark
js执行过程中会创建执行上下文，分为全局执行上下文以及函数执行上下文。
javascript是单线程，每次创建执行上下文时候，这个上下文就会被推入一个环境栈中，我们称为这个栈叫做执行上下文栈，当执行上下文结束以后，就会被释放，并且销毁，并且销毁执行上下文中的变量对象，继续执行其他上下文
```
- 每当执行流转到可执行代码时，即会进入一个执行环境。活动的执行环境构成一个栈：栈的底部始终是全局环境，顶部是当前活动的执行环境
- 全局执行环境是最外围的一个执行环境。在浏览器中，全局环境就是window对象，因此所有全局变量和函数都是作为window对象的属性和方法创建的
- 每个函数都有自己的执行环境。当执行流进入一个函数时，函数的环境被推入栈中。而在函数执行之后，栈将其环境弹出，把控制权返回给之前的执行环境。某个执行环境中的代码执行完后，该环境销毁，保存在其中的所有变量和函数定义也随之销毁。而全局执行环境直到应用程序退出才会被销毁（这里面会有垃圾回收机制问题）
- 

##### 执行上下文概念
在JavaScript解释器内部，每次调用Execution Context都会经历下面两个阶段：
- 创建阶段（发生在函数调用时，但是内部代码执行前，这将解释声明提升现象）
  - 创建作用域链
  - 创建变量对象VO
  - 确定this的值
- 激活/执行代码阶段
 - 变量赋值、执行代码


**创建阶段**
- Global Execution Context中没有这一步） 创建arguments对象，扫描函数的所有形参，并将形参名称 和对应值组成的键值对作为变量对象VO的属性。如果没有传递对应的实参，将undefined作为对应值。如果形参名为arguments，将覆盖arguments对象
- 扫描Execution Context中所有的函数声明（注意是函数声明，函数表达式不算）
  - 将函数名和对应值（指向内存中该函数的引用指针）组成组成的键值对作为变量对象VO的属性
  - 如果变量对象VO已经存在同名的属性，则覆盖这个属性
- 扫描Execution Context中所有的变量声明
  - 由变量名和对应值（此时为undefined） 组成，作为变量对象的属性
  - 如果变量名与已经声明的形参或函数相同，此时什么都不会发生，变量声明不会干扰已经存在的这个同名属性。


##### 作用域链 scope
> 作用域链本质上，是一个指向变量对象的指针列表，它只引用但不实际包含变量对象。  
> 注意重要的一点：[[scope]]属性在函数创建时被存储，永远不变，直到函数销毁。函数可以不被调用，但这个属性一直存在。
且，与作用域链相比，作用域链是执行环境的一个属性，而[[scope]]是函数的属性。

- 形参优先级高于当前函数名，低于内部函数名(函数提升最高优先级)
  ```javascript
    function fn(aa){
      console.log(arguments);// Arguments ["hello world"]
    }
    fn('hello world');

    function fn(arguments){
        console.log(arguments);// hello world
    }
    fn('hello world');

    function show(fn) {
      console.log(fn); //  11
      function fn () {}
      console.log(fn); //  22
    }
    show(111)
  ```
- 形参优先级高于arguments
- 形参优先级高于只声明却未赋值的局部变量，但是低于声明且赋值的局部变量
- 函数和变量都会声明提升，函数名和变量名同名时，函数名的优先级要高。执行代码时，同名函数会覆盖只声明却未赋值的变量，但是它不能覆盖声明且赋值的变量
- 局部变量也会声明提升，可以先使用后声明，不影响外部同名变量

#### 变量提升
主要原因在执行上下文中，创建变量对象(活动对象)会出现变量提升以及函数提升  
**注意形参也会在内存中开辟一个栈**
```javascript 
    /**
     * 
     *  老版浏览器（IE10以下）忽略“{}”的影响，继续声明/定义，不存在块级作用域
        新版浏览器中“{}”里的function只声明不定义，“{}”若出现funciton/let/const关键字，会创建一个块级上下文
     * 
     * @descript if语句中快级上下文，进入该快级上下文，创建活动对象a 
     *           最终快级a被赋值21，window的a仍是
     * */
    var a = 0;

    if (true) {
      a = 1;
      function a() {} // 执行这一步的时候，会在window上创建a a = 1。。。 后续a仍是改变快级作用域内部a
      a = 21;
      console.log(a); // 21 内部a
    }

    console.log(a); // 1


    /** 匿名函数内部自己的作用域，外部变量不可访问 */ 
    var a = 10;
    (function () {
        console.log(a) // undefined
        a = 5
        console.log(window.a) // 10
        var a = 20;  // 变量提升
        console.log(a)  // 20
    })()


    /**
     * 形参会开辟一个变量，指向了和window中a相同的内存地址，然后在修改对象增加了age属性，后续再改变形参的指针，使其指向了 {age: 21}
     * */
    var a = { name: 'levi' };
    function init(a) {
      a.age = 20;
      a = { age: 21 };

      console.log(a); // {age: 21}
    }

    init(a); // {name: "levi", age: 20}


    /**
      1、声明一个变量，为引用类型
        2和8、声明一个匿名函数，并立即执行，传递的参数是第1行中的foo。将一个对象类型赋值给一个新的变量，由于对象是引用类型，
        实质上是指将对象的地址赋值给该变量（也就是说这两个变量指向同一个地址空间），因此改变新的变量中的属性值或方法，对应的原来对象的值也会改变。
      3、原题中的第5行，由于存在变量提升，因此会在函数开始就声明，此时为undefined；然而由于一个变量的声明优先级低于形参，所以这行没有任何效果
      4、打印形参的foo.n,打印1
      5、改变第1行变量foo的属性n的值为3；
      6、重新声明并定义了一个变量，开辟了新的内存空间，n为2
      7、由于js中的代码是自上而下执行，所以此时输出2
      9、上面的函数调用结束后，局部变量被销毁，而之前的内存空间值已经变为3，所以输出3
     * */
    var foo = { n: 1 };
    (function (foo) {
      console.log(foo.n);  // 1
      foo.n = 3;
      var foo = { n: 2 };
      console.log(foo.n); // 2
    })(foo);
    console.log(foo.n); // 3

```

#### 普通函数与箭头函数的区别
- 语法更加清晰、简洁
- 箭头函数没有this,没有原型对象，因此。call/apply/bind 无法动态改变（运行时）改变箭头中this指向, this在定义时候已经确认
```javascript
  var id = 10; // 不能使用const id， 不会挂在window.id
  const fn = () => console.log(this.id);

  fn(); // 10
  fn.call({id: 20}); // 10
  fn.apply({id: 20}); // 10
  fn.bind({id: 20})(); // 10

```
- 不能作为构造函数被实例化
- 箭头函数没有arguments，取而代之是rest代替arguments对象，借此来访问参数序列
```javascript
  const levi_r = () => {
    console.log(arguments);
  };
  levi_r(1, 2, 3); // arguments is not undefined

  const levi_fn = (...rest) => {
    console.log(rest);
  };
  levi_fn(1, 2, 3, 4); // [1, 2, 3, 4]
```

- 箭头函数不能作为Generator函数，直接报错
```javascript
  function* levi() {
    yield 1;
    yield 2;
  }
  const itr = levi(); // 返回一个迭代器
  console.log(itr.next()); // {value: 1, done: false}

    const levi_arrowa = * () => {} // error 运行时报错，无法识别函数
    const * levi_arrowb = () => {} // error 运行时报错，无法识别函数
```

#### 闭包

闭包是存在自由变量的函数，即：闭包可以访问所在的词法作用域，并且拥有更长的生命周期，保持对上一层词法作用域的引用
闭包是有权访问一个函数内部变量的函数，说到底就是执行上下文执行结束之后，活动对象即变量对象没有被垃圾回收机制回收（存在自由变量的函数），被外部所引用，从而导致内存泄露
- 从理论角度：所有函数都是闭包。因为它们在创建的时候就将所有父环境的数据保存起来了。哪怕是简单的全局变量也是如此，
  因为在函数中访问全局变量就相当于在访问自由变量（指不在参数声明，也不在局部声明的变量），这个时候使用最外层的作用域
- 代码执行完毕后，所在的环境会被销毁，web中全局执行环境是window对象，全局环境会在应用程序退出时被销毁
- 从实践角度：以下函数才算是闭包：
  - 即使创建它的环境销毁，它仍然存在（比如，内部函数从父函数返回
  - 在代码中引用了自由变量

  ```javascript
  <!-- demo -->
  function init() {
    const str = 1;
    return () => str;
  }
  console.log(init()());

  <!-- demo -->
  var data = [];
  for (var k = 0; k < 3; k++) {
    data[k] = (function _helper(x) {
      return function () {
        alert(x);
      };
    })(k); // 传入"k"值
  }

  // 现在结果是正确的了
  data[0](); // 0
  data[1](); // 1
  data[2](); // 2

  <!-- demo -->
  const data = [];
  for (let k = 0; k < 3; k++) {
    data[k] = function () {
      console.log(k);
    }; // 传入"k"值
  }
  data[0](); // 0
  data[1](); // 1
  data[2](); // 2
  ```

#### 垃圾回收机制
js中垃圾回收机制是为了防止内存泄露，垃圾回收机制就是间歇的不定期的寻找到不再使用的变量，并释放掉它们所指向的内存
**垃圾回收机制运行在cup空闲的时候**
##### 场景
- 全局变量不能被垃圾回收机制回收，全局变量始终活跃在global执行环境中， 只有程序退出的时候才会退出
- 闭包引起的内存泄露
- 定时器造成的内存泄露，定时器可能会使用其他变量或者DOM元素。 解决办法：执行一段时间后需手动清除定时器

##### 标记清除
该算法假定设置一个叫做根（root）的对象（在Javascript里，根是全局对象）。垃圾回收器将定期从根开始（在JS中就是全局对象）扫描内存中的对象。凡是能从根部到达的对象，都是还需要使用的。
那些无法由根部出发触及到的对象被标记为不再使用，稍后进行回收。
此算法可以分为两个阶段，一个是标记阶段（mark），一个是清除阶段(sweep)。

- 标记阶段： 标记为可达对象，垃圾回收器会从根对象开始遍历。每一个可以从根对象访问到的对象都会被添加一个标识，于是这个对象就被标识为可到达对象。
- 清除阶段： 不定时进行先行遍历，垃圾回收器会对堆内存从头到尾进行线性遍历，如果发现有对象没有被标识为可到达对象，那么就将此对象占用的内存回收，并且将原来标记为可到达对象的标识清除，以便进行下一次垃圾回收操作。

标记清除算法缺陷
- 哪些无法从根查询的到对象将被清除
- 造成内存碎片化

##### 引用计数
> 跟踪并记录每个变量被引用的次数
统计引用类型变量声明后被引用的次数，当次数为 0 时，该变量将被回收
```javascript
function func4 () {
      const c = {} // 引用类型变量 c的引用计数为 0
      let d = c // c 被 d 引用 c的引用计数为 1
      let e = c // c 被 e 引用 c的引用计数为 2
      d = {} // d 不再引用c c的引用计数减为 1
      e = null // e 不再引用 c c的引用计数减为 0 将被回收
}
```

> 引用计数弊端
但是引用计数的方式，有一个相对明显的缺点——循环引用
```javascript
function func5 () {
      let f = {}
      let g = {}
      f.prop = g
      g.prop = f
      // 由于 f 和 g 互相引用，计数永远不可能为 0
}
```
像上面这种情况就需要手动将变量的内存释放
```javascript
  f.prop = null;
  g.prop = null;
```

主流浏览器中都是使用标记清除


##### 引用文章
- [垃圾回收机制算法](https://www.jianshu.com/p/a8a04fd00c3c)
- [掘金 javascript 垃圾回收机制](https://juejin.cn/post/6844903652331618312)
- [知乎 javascript 垃圾回收机制](https://zhuanlan.zhihu.com/p/60336501)
- [浅谈 javascript垃圾回收机制](https://zhuanlan.zhihu.com/p/60279001)

#### new 本质
- 创建一个空对象，并将空对象的原型指向了构造函数的原型对象
- 使用传入的参数，执行构造函数，并将构造函数的this改写为改空对象
- 构造函数执行完毕，存在返回值并且是一个对象，那么实例化的变量指向该对象，否则指向刚刚创建的空对象
```javascript
  function New(age, fn) {
    // const instance = Object.create(New.prototype);  方式1

    // 方式2
    const obj = {};
    obj.__proto__ = fn.prototype;

    const result = fn.call(instance, age);
    return typeof result === 'object' ? result : instance;
  }

  function Demo(age) {
    this.age = age;

    return [];
  }

  const a = New(20, Demo);
```

#### this 指向
this是运行的决定的，不是定义决定，es6中箭头函数除外，是定义的时候决定的。 
this指向调用它的对象，可能是对象也有可能是window(也是对象)
```javascript
    function Foo() {
      getName = function () {
        console.log(1);
      };
      return this;
    }
    Foo.getName = function () {
      console.log(2);
    };
    Foo.prototype.getName = function () {
      console.log(3);
    };
    var getName = function () {
      console.log(4);
    };
    function getName() {
      console.log(5);
    }

    Foo.getName(); // 2
    getName(); // 4
    Foo().getName(); // 1
    getName(); // 1
    new Foo.getName(); // 2

    new Foo().getName(); // 3   点操作大于() 所以相当于先执行了Foo.getName在执行 new操作
    new new Foo().getName(); //3
```
- 如果一个函数中有this，但是它没有被上一级的对象所调用，那么this指向的就是window，这里需要说明的是在js的严格版中this指向的不是window
- 如果一个函数中有this，这个函数有被上一级的对象所调用，那么this指向的就是上一级的对象
- 如果一个函数中有this，这个函数中包含多个对象，尽管这个函数是被最外层的对象所调用，this指向的也只是它上一级的对象
```javascript
    var obj = {
      a: 10,
      b: {
        a: 20,
        fn() {
          console.log(this.a);
        }
      }
    };
    window.obj.b.fn(); // 20 this指向了b
```
> 1.this永远指向最后调用的对象  
> 2. 构造函数内部的this指向了实例化对象

#### Promise/A+规范
```javascript  
    // 处理大量try catch造成的造成问题
    const request = async promise =>
      Promise.resolve(promise)
        .then(res => [null, res])
        .catch(err => [err, null]);

    async function init() {
      const data = await request(Promise.reject('eee'));
      console.log(data);
    }

    init();
```
##### 实现

```javascript
  /**
   * @description promise手动实现
   * */
  const isFunction = fn => typeof fn === 'function';
  const PENDING = 'PENDING';
  const FULLFILLED = 'FULLFILLED';
  const REJECTED = 'REJECTED';

  class LeviPromise {
    constructor(callback) {
      if (!isFunction(callback)) {
        throw new Error('this is not a function');
      }

      this._status = PENDING;
      this._value = undefined;
      this._fullFilledQueues = [];
      this._rejectedQueues = [];

      try {
        callback(this._resolve.bind(this), this._reject.bind(this));
      } catch (err) {}
    }

    _resolve(value) {
      const run = () => {
        if (this._status !== PENDING) return;

        const runFullFilled = value => {
          let cb;
          while ((cb = this._fullFilledQueues.shift())) {
            cb(value);
          }
        };

        const runRejected = value => {
          let cb;
          while ((cb = this._rejectedQueues.shift())) {
            cb(value);
          }
        };

        /** 构造函数内部resolve promise
         *  new LeviPromise(resolve => resolve(LeviPromise.resolve(2))).then(res => console.log(res));
         *  */
        if (value instanceof LeviPromise) {
          value.then(
            res => {
              this._value = res;
              this._status = FULLFILLED;
              runFullFilled(value);
            },
            err => {
              this._value = err;
              this._status = REJECTED;
              runRejected(err);
            }
          );
        } else {
          this._status = FULLFILLED;
          this._value = value;
          runFullFilled(value);
        }
      };

      setTimeout(() => run(), 0);
    }

    _reject(err) {
      if (this._status !== PENDING) return;

      const run = () => {
        this._status = REJECTED;
        this._value = err;
        let cb;
        while ((cb = this._rejectedQueues.shift())) {
          cb(err);
        }
      };

      setTimeout(() => run(), 0);
    }

    then(onFullFilled, onRejected) {
      const { _value, _status } = this;

      /**
       *  @desc: 最终思想，
       *  then 方法返回一个promise,并且将回调函数加入事件队列中，定时器过后拿出队列的回调函数执行。
       *   如果函数有返回值，则需要将返回值传入下一个promise的 onFullFilledfn中（resolve方法）
       * */
      return new LeviPromise((onFullFilledfn, onRejectedFn) => {
        const fullFilled = value => {
          try {
            if (!isFunction(onFullFilled)) {
              onFullFilledfn(value);
              return;
            }

            const res = onFullFilled(value);
            if (res instanceof LeviPromise) {
              res.then(onFullFilledfn, onRejectedFn);
            } else {
              onFullFilledfn(res); // 当前第二个promise的resolve方法
            }
          } catch (err) {
            onRejectedFn(err);
          }
        };

        const reject = err => {
          try {
            if (!isFunction(onRejected)) {
              onRejectedFn(err);
              return;
            }

            const res = onRejected(err);
            if (res instanceof LeviPromise) {
              res.then(onFullFilledfn, onRejectedFn);
            } else {
              onFullFilledfn(res);
            }
          } catch (err) {
            onRejectedFn(err);
          }
        };

        switch (_status) {
          case PENDING:
            this._fullFilledQueues.push(fullFilled);
            this._rejectedQueues.push(reject);
            break;
          case FULLFILLED:
            fullFilled(_value);
            break;
          case REJECTED:
            reject(_value);
            break;
        }
      });
    }

    static resolve(value) {
      if (value instanceof LeviPromise) return value;
      return new LeviPromise(resolve => resolve(value));
    }

    static reject(value) {
      if (value instanceof LeviPromise) return value;
      return new LeviPromise(resolve => resolve(value));
    }

    static all(list = []) {
      return new LeviPromise((resolve, reject) => {
        const values = [];
        let count = 0;

        for (let [i, p] of list.entries()) {
          console.log(i);
          LeviPromise.resolve(p).then(
            res => {
              values[i] = res;
              count++;

              if (count === list.length) {
                resolve(values);
              }
            },
            err => {
              reject(err);
            }
          );
        }
      });
    }

    static race(list = []) {
      return new LeviPromise((resolve, reject) => {
        for (let [i, p] of list.entries()) {
          console.log(i);
          LeviPromise.resolve(p).then(
            res => {
              resolve(res);
            },
            err => {
              reject(err);
            }
          );
        }
      });
    }

    finally(cb) {
      if (!isFunction(cb)) {
        throw new Error('this is not a functiom');
      }

      return this.then(
        res =>
          LeviPromise.resolve(cb()).then(() => {
            return res;
          }),
        err =>
          LeviPromise.reject(cb()).then(() => {
            throw new Error(err);
          })
      );
    }
  }

  LeviPromise.race([
    LeviPromise.resolve('a'),
    new LeviPromise((resolve, reject) => reject('err')),
    LeviPromise.resolve('b')
  ]).then(
    res => console.log(res),
    err => console.log(err)
  );
```

#### ES6
> [阮一峰es6](https://es6.ruanyifeng.com/#docs/set-map)
涵盖点
- let const / 解构赋值
- 字符串扩展
  - 字符串迭代器接口 可被for of 消费
  - 模板字符串
  - String.prototype.indexOf / String.prototype.includes /  String.prototype.startsWith /  String.prototype.endsWith
  -  String.prototype.repeat /  String.prototype.padStart /  String.prototype.padEnd
  -  String.prototype.trimStart /  String.prototype.trimEnd 
  -  String.prototype.replace /  String.prototype.replaceAll
- 数值扩展
- 函数扩展
  - 箭头函数
  - 剩余参数 function (a = 1, ...b) {}
  - 函数参数带有默认值 eg: a = 1
  - function.length / function.name
- 数组扩展 
  - Array.from() / Array.of /  Array.find / Array.findIndex / Array.fill / 
  - Array.keys / Array.values / Array.entries /
  - Array.includes / Array.flat / 
  - 扩展运算符
- 对象扩展
  - 扩展对象/解构赋值
  - 属性名简写、属性名表达式obj['a' + 'bc'] = 123;
  - 链判断运算符 obj?.info?.name
  - null 运算符 var b  = undefined ?? 12
  - Object.is 、 Object.getPropertypeOf 、 Object.setPropertyOf()
  - Object.keys 、 Object.values 、 Object.entries 、 Object.fromEntries(map: Map) => object
- Symbol / Symbol.for
- Proxy/Reflect
- class
- module加载实现
- set
  - 基本方法
    - set.add(x)
    - set.has(x)
    - set.delete(x)
    - set.clear() 清空  
    - set.size
  - 其他方法
    - set.keys() 
    - set.values()
    - set.entries()
    - Set.prototype[Symbol.Iterator] === Set.prototype.values 返回的是一个迭代器
  - set 转化为数组
    - Array.from(set) 
    - [...set]
- weakSet
  - 接受数组并且数组内部成员是 数组或者对象 new WeakSet([{}, []])
  - 方法
   - weakSet.add(x)
   - weakSet.has(x)
   - weakSet.delete(x)
- map new Map([['key', 'value'], ['key', 'value']])
  - 基本方法
   - map.set(x)
   - map.get
   - map.has
   - map.delete
   - map.clear
   - map.size
  - 其他方法
   - map.keys()
   - map.values()
   - map.entries()
 - 转化为数组
  - [...map]
  - [...map.keys()]
  - [...map.values()]
  - [...map.entries()]
- weakMap const wm2 = new WeakMap([[object1, 'foo'], [object2, 'bar']]);
  - WeakMap只接受对象作为键名（null除外），不接受其他类型的值作为键名
  - 基本方法
   - weakMap.set()
   - weakMap.get()
   - weakMap.has()
   - weakMap.delete()

#### 用Array的 reduce实现map
```javascript
    Array.prototype.LMap = function (fn, thisArg) {
      return this.reduce((prev, cur, index, arr) => {
        prev.push(fn.call(thisArg, arr[index]));
        return prev;
      }, []);
    };

    const a = [1, 2, 3, 4];
    console.log(a.LMap(item => item * 2)); // 2,4,6,8

    Array.prototype.LMap = function (fn, args) {
      return this.reduce((prev, cur, index, arr) => {
        prev[index] = fn.call(args, cur, index);
        return prev;
      }, []);
    };

    Array.prototype.LFilter = function (fn, args) {
      return this.reduce((prev, cur, index, arr) => {
        if (fn.call(args, cur, index)) {
          prev.push(cur);
        }
        console.log(cur);

        return prev;
      }, []);
    };

    Array.prototype.LFindInIndex = function (fn, args) {
      return this.reduce((prev, cur, index) => {
        return prev > -1 ? prev : fn.call(args, cur, index) ? index : -1;
      }, -1);
    };

    Array.prototype.LFinedLastIndex = function (fn, args) {
      const indexList = [];
      this.reduce((prev, cur, index, arr) => {
        if (fn.call(args, cur, index)) {
          indexList.push(index);
        }
      }, []);

      return indexList.length ? indexList[indexList.length - 1] : -1;
    };
```

#### 设计模式
可分为以下几种
- 单例模式
- 发布订阅模式
- 观察者模式
- ....

> 观察者模式是面向目标(subject)和观察者(observer)编程，而发布订阅则是面向调度中心编程， 观察者用于耦合目标和观察者，发布订阅则是用于松耦发布者和订阅者
##### 发布订阅模式
发布订阅模式比观察者模式多一个事件调度中心，管理事件的订阅以及发布，隔绝了发布者和订阅者的依赖关系，即订阅者在订阅事件的时候，只关注事件本身，无需关注谁发布这个事件；
发布者在发布事件的时候，同样只需关注事件本身，无需关注该事件被谁订阅了。  
 举个例子。你在微博关注了蔡徐坤。同时也有很多人关注他，当蔡徐坤发布一条微博，微博就会他发布这条消息。你也会受到这个消息的推送。这个流程中。 蔡徐坤就是发布者，微博平台就是事件调度中心。你就是订阅者。实现发布者和订阅者的松耦合
<img src="https://user-gold-cdn.xitu.io/2019/12/12/16ef7fe5614d6ea0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">

```javascript
 class PubSub {
      constructor() {
        this.events = {};
      }

      subcribe(type, callback) {
        if (!type) {
          throw new Error('please enter subscribe type');
        }

        if (!this.events[type]) {
          this.events[type] = [];
        }

        this.events[type].push(callback);
      }

      publish(type, ...args) {
        if (!type || !this.events[type]) {
          return false;
        }

        this.events[type].forEach(event => {
          event(...args);
        });
      }

      unSubscribeByFn(type, callback) {
        if (!type || !callback || !this.events[type]) {
          return false;
        }

        const index = this.events[type].findIndex(event => event === callback);
        this.events[type].splice(index, 1);
      }

      unSubscribeAll(type) {
        if (type) {
          this.events[type] = [];
        } else {
          this.events = {};
        }
      }
    }

    const pubsub = new PubSub();
    const leviRun = name => {
      console.log(`${name} is running`);
    };

    pubsub.subcribe('run', name => {
      console.log(name);
    });

    pubsub.subcribe('run', name => {
      console.log(name);
    });

    pubsub.subcribe('run', leviRun);

    pubsub.publish('run', 'levi'); // levi / levi /  levi is running

    pubsub.subcribe('getScore', score => {
      console.log(`you score: ${score}`);
    });

    pubsub.unSubscribeByFn('run', leviRun);
    pubsub.publish('run', 'levi'); // levi / levi

    pubsub.unSubscribeAll('run');

    pubsub.publish('run', 'levi'); // false
    pubsub.publish('getScore', 20); // you score: 20
```

##### 观察者模式 
观察者有2个重要角色。即：目标和观察者，在目标和观察者质检没有事件调度通道，一方面观察者如果想要订阅目标事件，需要把自己添加至目标(subject)进行管理。
另一方面，目标在触发事件，也无法将通知操作委托给事件调度中心，因此只能亲自通知观察者， 例如levi创建了一个公众号，文章重量极高，多个用户关注了”前端xxx“公众号，这些用户会定期收到该公众号的文章推送， levi的公众号被称为主体(Subject)，他的公众号粉丝称为订阅者(Observer)
<img src="https://user-gold-cdn.xitu.io/2019/12/12/16ef7fe567bdf007?imageView2/0/w/1280/h/960/format/webp/ignore-error/1">

```javascript
    class Subject {
      constructor() {
        this.observers = [];
      }

      addObserver(observer) {
        this.observers.push(observer);
      }

      removeObserver(observer) {
        const targetIndex = this.observers.findIndex(obs => obs.id === observer.id);
        if (!targetIndex !== -1) {
          this.observers.splice(targetIndex, 1);
        }
      }

      removeObserverByIndex(index) {
        this.observers.splice(index, 1);
      }

      notify(bookname) {
        this.observers.forEach(observer => {
          observer.update(bookname);
        });
      }
    }

    class Observer {
      constructor(id, name) {
        this.id = id;
        this.name = name;
      }

      update(title) {
        console.log(`我是${this.name}, 接受levi更新: ${title}`); 
      }
    }

    const leviSubject = new Subject();

    const jack = new Observer(1, 'jack');
    const mark = new Observer(2, 'mark');

    leviSubject.addObserver(jack);
    leviSubject.addObserver(mark);

    leviSubject.notify('观察者模式详解'); // 我是jack, 接受levi更新: 观察者模式详解 、 我是mark, 接受levi更新: 观察者模式详解

    leviSubject.removeObserver(jack);
    leviSubject.notify('发布订阅详解'); // 我是mark, 接受levi更新: 发布订阅详解
```
 
 参考文章：
 - [观察者模式和发布订阅模式有什么不同？](https://www.zhihu.com/question/23486749)

#### 排序
const leviArr = [1, 3, 4, 6, 2, 4, 6];
##### 冒泡排序
```javascript
  const leviArr = [1, 3, 4, 6, 2, 4, 6];
  function init(arr) {
    for (let i = 0; i < arr.length - 1; i++) {
      for (let j = 0; j < arr.length - 1 - i; j++) {
        if (arr[j] >= arr[j + 1]) {
          let temp = arr[j + 1];
          arr[j + 1] = arr[j];
          arr[j] = temp;
        }
      }
    }

    return arr;
  }

  // [1, 2, 3, 4, 4, 6, 6]
```
##### 选择排序
```javascript
function init(arr) {
  var minIndex, temp;
  for (let i = 0; i < arr.length - 1; i++) {
    minIndex = i;
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[j] <= arr[minIndex]) {
        minIndex = j;
        console.log(j);
      }
    }

    temp = arr[i];
    arr[i] = arr[minIndex];
    arr[minIndex] = temp;
  }
  return arr; // [1, 2, 3, 4, 4, 6, 6, 6]
}
```

##### 快速排序
```javascript
const leviArr = [7, 4, 2, 8, 7, 6, 1, 9, 5];

function quickSort(arr, left, right) {
  var _left = left || 0;
  var _right = right || arr.length - 1;

  if (_left < _right) {
    const middleIndex = getIndex(arr, _left, _right);
    quickSort(arr, _left, middleIndex - 1);
    quickSort(arr, middleIndex + 1, _right);
  }

  return arr;
}

/** 重点跟随指针移动  没交换一次，指针一栋一格， 最后指针后退一格，交换指针元素与首部元素。 最终结果指针右侧比当前元素大，指针左侧比当前元素小 */
function getIndex(arr, left, right) {
  const value = arr[left];
  let index = left + 1;

  for (let i = index; i <= right; i++) {
    if (arr[i] <= value) {
      withTemp(arr, i, index);
      index++; // 交换值指针前进
    }
  }

  withTemp(arr, left, index - 1); // 前后交换指针
  return index - 1;
}

function withTemp(arr, i, j) {
  let temp = arr[i];
  arr[i] = arr[j];
  arr[j] = temp;
}

console.log(quickSort(leviArr));
```

#### 去重
##### newSet
```javascript
  const leviArr = [1, 2, 3, 4, 5, 6];
  console.log(Array.from(new Set(leviArr))); // 1,2,3,4,5,6
```

##### arr reduce && includes
```javascript
function init(arr) {
  return arr.reduce((prev, cur) => {
    if (!prev.includes(cur)) {
      prev.push(cur);
    }

    return prev;
  }, []);
}

init(leviArr); // 1,2,3,4,5,6
```

##### indexOf
```javascript
function init(arr) {
  const result = [];
  for (let i = 0; i < arr.length; i++) {
    if (i === arr.indexOf(arr[i])) {
      result.push(arr[i]);
    }
  }

  return result;
}
```

##### filter && 对象属性值判断
```javascript
function init(arr) {
  const obj = {};
  return arr.filter(item => {
    return obj[item + item] ? false : (obj[item + item] = 1);
  });
}
```

#### 高级函数

- [x] componse
- [x] 柯里化
- [x] 节流
- [x] 防抖
- [x] 深拷贝 && 浅拷贝
- ...

##### 防抖
> 事件在指定N秒后执行，如果N秒期间在被触发，则重新计时
- 一般用于相应用户输入，多次提交等

```javascript
    function print(value) {
      console.log(value);
    }

    const debounce = function (fn, delay = 500) {
      if (typeof fn !== 'function') {
        throw new Error('this is not a function');
      }
      
      let timer;

      return function () {
        // const _this = this; 不需要引用this,setTimeout使用箭头函数
        if (timer) {
          clearTimeout(timer);
        }

        timer = setTimeout(() => {
          fn(...arguments);
        }, delay);
      };
    };
    const debouncePrint = debounce(print);

    const ele = document.getElementById('input');
    ele.addEventListener('keyup', e => {
      debouncePrint(e.target.value);
    });
```

##### 节流
> 在指定时间内触发事件一次
- 鼠标/触摸屏的mouseover/touchmove事件
- 页面窗口的resize事件
- 滚动条的scroll事件
```javascript
    function print(value) {
      console.log(value);
    }

    const throttle = function (fn, delay = 2000) {
      if (typeof fn !== 'function') {
        throw new Error('this is not a function');
      }

      let isStart = false;

      return function () {
        if (isStart) {
          return;
        }
        isStart = true; // 初始化设置start 要执行一次

        /**
         * @description 利用定时器回调来控制start流转状态
         * */
        setTimeout(() => {
          fn(...arguments);
          isStart = false;
        }, delay);
      };
    };
    const throttlePrint = throttle(print);

    const ele = document.getElementById('input');
    ele.addEventListener('keyup', e => {
      throttlePrint(e.target.value);
    });
```

##### 柯里化
```javascript
  function curry(fn, length) {
    const l = length || fn.length;

    return function method(...args) {
      if (args.length >= l) {
        fn.call(this, ...args);
      } else {
        return curry(fn.bind(this, ...args), l - args.length); // bind 返回一个新函数，接收了之前参数
      }
    };
  }

  const sum = curry(function (a, b, c) {
    console.log(a, b, c);
  });

  sum(1, 2)(3); // 1, 2, 3
  sum(1)(2)(3); // 1, 2, 3
```

```javascript
    // 柯里化
    const fn = x => y => z => j => k => x + y + z + j + k;

    /**
     * @description 每次执行完curry函数后，返回一个新的函数, 判断入参是否达到函数形参个数， 否则不断柯里化返回函数，最后满足条件，执行fn
     *
     * 如果存在n个参数一起执行 eg: const fn = x => y => z => j => k => ...... => x+y+z+....+n   写一个通用函数
     * */
    function sub_curry(fn) {
      const args = Array.prototype.slice.call(arguments, 1);

      // 返回函数，被调用直接执行，接受返回过来的参数
      return function () {
        fn.apply(this, [...arguments, ...args]);
      };
    }

    function curry(fn, length) {
      const l = length || fn.length;

      return function () {
        if (arguments.length < l) {
          const combined = [fn].concat(Array.prototype.slice.call(arguments));

          return curry(sub_curry.apply(this, combined), l - arguments.length);
        } else {
          return fn.apply(this, arguments);
        }
      };
    }

    var method = curry(function (a, b, c) {
      console.log(a + b + c);
    });

    method(1, 2, 3); // 6
    method(1, 2)(3); // 6
    method(1)(2)(3); // 6
```

##### compose
> compose 数据流从右往左流转
```javascript
  function add1(str){
    return str+='1';
  }
  function add2(str){
    return str+='2';
  }
  // 想要str先加上 '1'再加上'2',一般这么写
  add2(add1(str))

  // 但很多层之后，可读性会很差,于是想到组合函数，将要处理的功能组合成一个函数，再传入一个参数即可。
  compose(add1,add2)(str)

  // 从左到右，从右到左的话，for那边改为--之类的即可
  function compose(...rest) {
    return function(str) {
      var val = str;
      for (var i = rest.length - 1; i > -1; i--) {
        val = rest[i](val);
      }
      return val
    }
  }

  // 升级版，同样从右到左的话，reduceRight,这个简直了,这个不记得从哪看到的，不是原创
  function compose(...rest) {
    return str => rest.reduce((params, cb) => cb(params), str);
  }

```
**redux compose补充** 
compose从右往左执行
```javascript
  export default function compose(...funcs) {
    if (funcs.length === 0) {
      return arg => arg
    }

    if (funcs.length === 1) {
      return funcs[0]
    }
    return funcs.reduce((a, b) => (...args) => a(b(...args)))
  }


  import {compose} from 'redux'
  let x = 10
  function fn1 (x) {return x + 1}
  function fn2(x) {return x + 2}
  function fn3(x) {return x + 3}
  function fn4(x) {return x + 4}

  // 假设我这里想求得这样的值
  let a = fn1(fn2(fn3(fn4(x)))) // 10 + 4 + 3 + 2 + 1 = 20

  // 根据compose的功能，我们可以把上面的这条式子改成如下：
  let composeFn = compose(fn1, fn2, fn3, fn4)
  let b = composeFn(x) // 理论上也应该得到20
```

##### 深拷贝 
```javascript
  const getType = obj => Object.prototype.toString.call(obj).slice(8, -1);

  function deepClone(target) {
    const type = getType(target);
    if (!['Array', 'Object'].includes(type)) {
      return target;
    }

    const result = type === 'Array' ? [] : {};
    for (let key in target) {
      if (Object.prototype.hasOwnProperty.call(target, key)) {
        if (type === 'Array') {
          result.push(deepClone(target[key]));
        } else {
          result[key] = deepClone(target[key]);
        }
      }
    }

    return result;
  }
```
克隆对象改变不会影响原对象
```javascript 递归调用
    const person = {
      info: {
        name: 'levi',
        age: 20
      },
      company: 'ahs',
      address: null
    };

    const arr = [1, 2, person];

    function getObjectType(obj) {
      return Object.prototype.toString.call(obj).slice(8, -1);
    }

    const _deepClone = obj => {
      const type = getObjectType(obj);
      const target = type === 'Array' ? [] : {};

      for (let key in obj) {
        if (!obj.hasOwnProperty(key)) {
          continue;
        }

        if (['Array', 'Object'].includes(getObjectType(obj[key]))) {
          target[key] = _deepClone(obj[key]);
          continue;
        }

        target[key] = obj[key];
      }

      return target;
    };

    const newObj = _deepClone(person);
    const newArr = _deepClone(arr);

    newObj.company = 'aaa';
    newObj.info.age = 30;

    newArr[0] = 100;
    newArr[2].company = 'blibli';
    newArr[2].info.age = 40;

    console.log(person, newObj);
    console.log(arr, newArr);
```

##### 浅拷贝
浅拷贝会造成对象容易受到克隆对象影响
- 直接赋值  = 
- Object.assign()
- es6 扩展运算符  const a = {...obj}
- JSON.parse(JSON.stringfy(obj))

#### 事件委托 &&  DOM事件捕获、目标、冒泡
事件模型中分为： 事件捕获、目标以及事件冒泡阶段。  先开始事件捕获 => 目标 => 事件冒泡

> 获取当前事件目标 e.target or e.srcElement

```javascript
  xxx.addEventListener([事件名称], 方法, 冒泡or捕获)

  // 事件名称： click, mouseenter等
  // 方法： 函数
  // 冒泡or捕获： 传true 则代表是捕获阶段出发， 传false 或者不传代表冒泡阶段出发
```

```javascript
    const btn = document.getElementById('btn');
    const container = document.getElementById('container');

    document.body.addEventListener('click', e => console.log('body', '捕获', e), true);
    document.body.addEventListener('click', e => console.log('body', '冒泡', e), false);

    container.addEventListener('click', e => {
      console.log(`container`, '冒泡', e);
    });
    container.addEventListener(
      'click',
      e => {
        console.log(`container`, '捕获', e);
      },
      true
    );

    /** 冒泡阶段 */
    btn.addEventListener('click', e => {
      console.log(`btn1`, e);
      console.log(e.target, e.srcElement, e.target === e.srcElement); // btn, btn  true
      // e.stopPropagation();
      // e.stopImmediatePropagation(); // 自身的事件都会组织，例如 btn2 不会输出
    });

    btn.addEventListener('click', e => {
      console.log(`btn2`, e);
    });

    /***
      执行顺序
      1、body 捕获
      2. container 捕获
      3. btn 第一次点击
      4. btn第二次点击
      5. container 冒泡
      6. body 冒泡
    */
```

<div style="text-align: center">
  <img  src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee731604bc4e4c0d8b693a212684c20e~tplv-k3u1fbpfcp-watermark.image" style="width:400px"/>
</div>

- ##### 事件委托
事件委托利用事件冒泡机制，将子元素事件挂在父元素上绑定函数
如下面实例，不利用事件委托的话，要给所有li添加方法的话，会循环遍历所有li，给每个li添加函数，这样会这样内存消费过大，并且动态li，也要重新绑定事件，可维护性差

> 事件委托： 节省内存，可读性强
```javascript

    // <ul id="ul">
    //   <li>1111</li>
    //   <li>2222</li>
    //   <li>3333</li>
    //   <li>4444</li>
    //   <li>5555</li>
    // </ul>
    
    const li = document.getElementsByTagName('li');
    const ul = document.getElementById('ul');

    for (let i = 0; i < li.length; i++) {
      li[i].onclick = function () {
        console.log(li[i].innerText);
      };
    }

    //  事件委托
    ul.addEventListener(
      'click',
      e => {
        console.log(e.target.innerText);
      },
      true
    );

```

#### 事件循坏

浏览器维护了一个事件队列，当主线程任务执行完成后，每隔相应的时间就会来事件队列查看，看看有咩有需要执行的任务。任务又分为宏任务和微任务，宏任务快于微任务

> 宏任务放入宏任务队列
> 微任务放入微任务队列

- 常见的宏任务：script（整体代码）、setTimeout、setInterval、I/O、setImmedidate，ajax
- 常见的微任务：process.nextTick、MutationObserver、Promise.then catch finally

**执行宏任务先检测是否有微任务，有微任务的话，先执行微任务，在执行宏任务**

<div style="text-align:center">
  <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ee050b7135b0428e83d9bb696c1d1d90~tplv-k3u1fbpfcp-zoom-1.image">

---

  <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d0408fa7236b45b78a5eae7361f42f71~tplv-k3u1fbpfcp-zoom-1.image">
</div>

整体执行如上图所示

1. 按照代码加载顺序执行，遇到宏任务放入宏任务队列，遇到微任务放入微任务队列， 同步执行代码块立即执行
2. 先执行一个宏任务, 检查是否有微任务，全部执行。（微任务产生的微任务也会全部执行）,然后清空微任务队列
3. 渲染 - 开启下一个宏任务 ，重复步骤 2

```javascript
setTimeout(() => {
  console.log('timer1');

  Promise.resolve().then(function () {
    console.log('promise1');
  });
}, 0);
// 简称set2
setTimeout(() => {
  console.log('timer2');

  Promise.resolve().then(function () {
    console.log('promise2');
  });
  // 简称set3
  setTimeout(() => {
    console.log('timer3');
  }, 0);
}, 0);

Promise.resolve().then(function () {
  console.log('promise3');
});

console.log('start');

/***
 @description 输出结果

 1. start
 2. promise3
 3. timer1 
 4. promise1
 5. timer2
 6. promise2
 7. timer3
 
**/



  async function async1() {
    console.log('async1 start');
    await async2();
    console.log('async1 end');
  }

  async function async2() {
    console.log('async2');
  }

  async1();

  new Promise(function (resolve) {
    console.log('promise1');
    resolve();
  }).then(function () {
    console.log('promise2');
  });
  console.log('script end');

  /**
    @description 输出
    1. async1 start
    2. async2
    3. promise1
    4. script end 
    5. async1 end
    6. promise2

    async await 相当于 Promise的语法糖
    async1 () {
      console.log('async1 start');
      Promise.resolve(async2()).then(() => console.log(async1 end))
    }
    async1();
  */ 

```

#### requestAnimationFrame 与 requestIdleCallback 的区别
- requestAnimationFrame在浏览器的运行的每一帧都会执行，属于高优先级任务
- requestIdleCallback里面注意的回收的回调在浏览器每一帧空余的时间才会执行， 同时可以设置一个超时时间
> window.requestAnimationFrame() 告诉浏览器——你希望执行一个动画，并且要求浏览器在下次重绘之前调用指定的回调函数更新动画。
> 该方法需要传入一个回调函数作为参数，该回调函数会在浏览器下一次重绘之前执行

requestAnimationFrame 比起 setTimeout、setInterval 的优势主要有两点：
- requestAnimationFrame 会把每一帧中的所有 DOM 操作集中起来，在一次重绘或回流中就完成，并且重绘或回流的时间间隔紧紧跟随浏览器的刷新频率，一般来说，这个频率为每秒 60 帧。
- 在隐藏或不可见的元素中，requestAnimationFrame 将不会进行重绘或回流，这当然就意味着更少的的 cpu，gpu 和内存使用量。

```javascript

  requestAnimationFrame(callback);

  requestIdleCallback(callback, {timeout: xxx})

```

#### esmodule 和 commonjs 的区别

#### TS 的 type 和 interface 的区别
相同点
- 都可以描述Object或者Function
  ```javascript
    type Person = {
      name: string;
      age: number;
    };

    type Person = () => Promise<boolean>;

    type Person = string | number;


  interface Levia {
    name: string;
    age: number;
    fn: () => string;
  }

  interface ILevi {
    (a: number, b: string): string;
  }

  ```
- 都可以被继承，使用泛型

    ```javascript

    type Base = {
      name: string;
    };

    interface IBase {
      name: string;
    }

    type Person = Base & IBase & { age: number };

    interface ILevi extends Base, IBase {
      say: () => void;
    }

  ```
- 都可以被类实现<implements>, 注意类不能实现联合类型的继承
  ```javascript

    type Base = {
      name: string;
      say: () => void;
    };
    class Nie implements Base {
      name: string = 'levis';

      say() {
        console.log(`我是${this.name}`);
      }
    }

    interface IBase {
      age: number;
      run: () => string;
    }

    class levis implements IBase {
      public age: number = 20;
      public run() {
        return `我今年${this.age}`;
      }
    }

    console.log(new Nie().say()); // 我是levis
    console.log(new levis().run()); // 我今年20
  ```

不同点
- type 可以定义类型别名、
  ```javascript
    type UserName = string;
    type Studo = number;

    const a: UserName = 'levis';
  ```
-  type 可以声明联合类型/元祖类型，interfce不行
  ```javascript

    type Name =  string | number;

    type Person = ['a', 'b'];
    const Levi: Person = ['a', 'b'];

  ```

  - interface 会声明合并， type 声明合并会报错
  ```javascript

    interface IPerson {
      name: string;
    }

    interface IPerson {
      age: number;
    }

    export const levis: IPerson = {
      name: 'levis',
    };


    type Student = {
      name: string;
    };

    type Student = {
      age: number;
    };

    //  报错了： Duplicate identifier 'Student'.
    export const jack: Student = {
      name: 'levis',
      age: 20,
    };

  ```



####  typescript infer举例

```javascript

// 获取指定类型
type Foo<T> = T extends { a: infer U; b: infer U } ? U : never;
type Levi = Foo<{ a: string; b: string }>; // string


// 获取函数返回的promise类型
const fna = () => {
  return Promise.resolve(1);
};
const method = async () => 'levis'

type Foo<T extends (...arg: any) => Promise<any>> = T extends (...args) => Promise<infer U> ? U : never;
type Levia = Foo<typeof fn>; // number
type Levib = Foo<typeof method>; // string

// 获取函数的形参类型

const fn = async (a: number, b: number) => a + b;
const method = async (a: number, b: string) => a + b;

type Foo<T> = T extends (...args: (infer T)[]) => Promise<U> ? T : never;
type Levi = Foo<typeof fn>; // number;
type Levi = Foo<typeof method>; // number | string;

```