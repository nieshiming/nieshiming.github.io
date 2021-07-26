---
title: ByteDance八股文
date: 2021-07-23 16:31:28
tags: 'js'
categories: 'js'
---

待描述内容xxxxxxxxx 描述
<!--more-->

#### 实现lazyman
```javascript
 class LazyManClass {
      constructor(name) {
        this.taskList = [];
        this.name = name;
        this.taskList.push(() => {
          console.log(`Hi! This is ${name}!`);
          this.run();
        });
        setTimeout(() => {
          this.run();
        }, 0);
      }
      sleepFirst(time) {
        let fn = () => {
          setTimeout(() => {
            console.log(`先睡个${time}秒...`);
            this.run();
          }, time * 1000);
        };
        this.taskList.unshift(fn);
        return this;
      }
      eat(name) {
        let fn = () => {
          console.log(`I am eating ${name}`);
          this.run();
        };
        this.taskList.push(fn);
        return this;
      }
      sleep(time) {
        let fn = () => {
          setTimeout(() => {
            console.log(`睡了${time}秒...`);
            this.run();
          }, time * 1000);
        };
        this.taskList.push(fn);
        return this;
      }
      run() {
        let fn = this.taskList.shift();
        fn && fn();
      }
    }

    function LazyMan(name) {
      return new LazyManClass(name);
    }
    LazyMan('Tony').eat('lunch').eat('dinner').sleepFirst(2).sleep(10).eat('junk food');
```

#### Jquery选择器怎么实现的
```javascript
    (function (window, undefined) {
      var rootjQuery = window.document;
      var jQuery = (function () {
        var jQuery = function (selector, context) {
          return new jQuery.fn.init(selector, context, rootjQuery);
        };

        jQuery.fn = jQuery.prototype = {
          construct: jQuery,
          init: function (selector, context, rootjQuery) {
            var that = this;
            that.ele = null;
            that.value = '';
            if (selector.charAt(0) === '#') {
              console.log(selector);
              that.ele = document.getElementById(selector.slice(1));
            }
            that.getValue = function () {
              that.value = that.ele ? that.ele.innerHTML : 'No value';
              return that;
            };
            that.showValue = function () {
              return that.value;
            };
          }
        };

        jQuery.fn.init.prototype = jQuery.fn;
        return jQuery;
      })();

      window.jQuery = window.$ = jQuery;
    })(window);
```

#### 给页面注入50万个li怎么做提升性能？

#### 手写懒加载（考虑防抖和重复加载问题）
```javascript
    class lazyImage {
      constructor(selector) {
        this._throttleFn = null;
        this.imageElements = Array.prototype.slice.call(document.getElementsByTagName(selector));

        this.init();
      }

      init() {
        this.initShow();
        this._throttleFn = this.throttle(this.initShow);
        window.addEventListener('scroll', this._throttleFn.bind(this));
      }

      initShow() {
        let len = this.imageElements.length;
        for (let i = 0; i < len; i++) {
          let imageElement = this.imageElements[i];
          const rect = imageElement.getBoundingClientRect();
          // 出现在视野的时候加载图片
          if (rect.top < document.documentElement.clientHeight) {
            imageElement.src = imageElement.dataset.src;
            imageElement.removeAttribute('data-src');
            imageElement.style.display = 'block';
            Array.prototype.splice.call(this.imageElements, i, 1);
            len--;
            i--;

            if (this.imageElements.length === 0) {
              // 如果全部都加载完 则去掉滚动事件监听
              document.removeEventListener('scroll', this._throttleFn);
            }
          }
        }
      }

      throttle(fn, delay = 15) {
        if (typeof fn !== 'function') {
          throw new Error('this is not a function');
        }

        let isStart = false;

        return function () {
          const _this = this;
          if (isStart) {
            return;
          }
          isStart = true; // 初始化设置start 要执行一次

          /**
           * @description 利用定时器回调来控制start流转状态
           * */
          setTimeout(() => {
            fn.apply(_this, arguments); // 确定this指向当前调用者 input
            isStart = false;
          }, delay);
        };
      }
    }

    new lazyImage('img');
```

####  setTimeout和requestAnimationFrame的区别
setTimeout
- 需要设置时间间隔
- 间隔时间不精确，可能被其他任务阻塞
- 实现动画在指定时间后执行，无论页面是否可见，浪费系统资源
- 动画的间隔时间如果设定过短就会出现过度渲染占用大量资源，可能出现掉帧

requestAnimationFrame 
- 不需要设定时间间隔
- 元素不可见时不会引起回流或重绘
- 按帧对网页进行重绘。该方法告诉浏览器希望执行动画并请求浏览器在下一次重绘之前调用函数更新动画。
- 由系统来决定回调函数的执行机制, 会把回调函数加入到事件队列中在运行时浏览器会自动优化方法的调用。
- 可返回 animationId, 用于注销停止动画的执行  cancelAnimationFrame(RequestAnimationFrame(callback))
- 页面不是激活状态下的话，开启另一个tab,当前页开启requestAnimationFrame会停止,动画会自动暂停，有效节省了CPU开销

[https://juejin.cn/post/6844903761102536718](https://juejin.cn/post/6844903761102536718)

#### ajax和fetch区别
ajax 本质上是 XMLHttpRequest
```javascript
  var xhr = new XMLHttpRequest();
  xhr.open('GET', url);
  xhr.responseType = 'json';

  xhr.onload = function () {
    console.log(xhr.response);
  };

  xhr.onerror = function () {
    console.log('Oops, error');
  };

  xhr.send();
```

fetch window上的方法
- window的一个方法
- IE浏览器并不支持fetch
- 默认是get请求，可通过{method: 'post'}配置
- 默认不会接受或者发送cookies，需要设置 fetch(url, {credentials: 'include'})
- fetch请求不会讲http返回非200以外认为是错误状态，例如服务器返回 400，500 错误码时并不会 reject，只有网络错误这些导致请求不能完成时，fetch 才会被 reject

```javascript
  // 第一个参数是请求url
  // 第二个参数可选参数 可以控制不同的init对象
  // 默认返回一个promise
  var promise = fetch('http://localhost:3000/news', {
    method: 'get'
  })
    .then(function (response) {
      return response.json();
    })
    .catch(function (err) {
      // Error :(
    });
  promise
    .then(function (data) {
      console.log(data);
    })
    .catch(function (error) {
      console.log(error);
    });


  // 实例
  onfetch = () => {
    window.fetch('http://localhost:8081').then(res => console.log(res)).then;
    /**
      body: ReadableStream // 返回流，需要特定插件处理下
      bodyUsed: false
      headers: Headers {}
      ok: true
      redirected: false
      status: 200
      statusText: "OK"
      type: "cors"
      url: "http://localhost:8081/"
      __proto__: Response
     * */
  };


  //  node开启服务
  http
  .createServer((req, res) => {
    // const [path, query] = req.url.split('?')
    // const params = qs.parse(query, { ignoreQueryPrefix: true })
    console.log('开始请求了')
    res.writeHead(300, {
      'Content-Type': 'text/javascript',
      'Access-Control-Allow-Origin': '*',
      'Access-Control-Allow-Methods': 'PUT, GET, POST, DELETE, OPTIONS',
    })

    setTimeout(() => {
      res.end('levi')
    }, 3000)
  })
  .listen(8081)

```


#### 写一个加法函数(sum)，使他可以同时支持sum(x,y)和sum(x)(y)两种调用方式
```javascript
  function currying(func, ...args) {
    if (args.length >= func.length) {
      return func(...args);
    }
    return function (...args2) {
      return currying(func, ...args, ...args2);
    };
  }

  function add(num1, num2) {
    return num1 + num2;
  }
```

#### JS实现一个带并发限制的异步调度器Scheduler，保证同时运行的任务最多有两个。完善代码中Scheduler类，使得以下程序能正确输出
```javascript
// JS实现一个带并发限制的异步调度器Scheduler，保证同时运行的任务最多有两个。完善代码中Scheduler类，使得以下程序能正确输出。
    class Scheduler {
      constructor() {
        this.taskList = [];
        this.count = 0;
      }

      add(promiseCreate) {
        return new Promise(resolve => {
          this.taskList.push({
            creator: promiseCreate,
            resolve
          });

          this.run();
        });
      }

      run() {
        if (this.count >= 2) {
          return;
        }

        const task = this.taskList.shift();
        if (task) {
          this.count++;
          task.creator().then(res => {
            this.count--;
            task.resolve();
            this.run();
          });
        }
      }
    }

    const timerPromise = (timer = 1000) => {
      return new Promise(resolve =>
        setTimeout(() => {
          resolve();
        }, timer)
      );
    };

    const s = new Scheduler();

    const addTask = (timer, data) => {
      s.add(() => timerPromise(timer)).then(() => console.log(data));
    };
    addTask(1000, '1');
    addTask(500, '2');
    addTask(300, '3');
    addTask(400, '4');
    // output: 2 3 1 4
    // 一开始，1、2两个任务进入队列
    // 500ms时，2完成，输出2，任务3进队
    // 800ms时，3完成，输出3，任务4进队
    // 1000ms时，1完成，输出1
    // 1200ms时，4完成，输出4
```


#### 实现flat    [1,2,3,[4,5,[6,7]]] -> [1,2,3,4,5,6,7]
```javascript
// 递归
function flat(arr, prev) {
	let result = prev || []
  
  for(let i=0; i<arr.length; i++) {
  	if (Array.isArray(arr[i])) {
        result.concat(flat(arr[i], result))
    } else {
    	result.push(arr[i])
    }
  }
  
  return result
}

//  递归 reduce
function flat (arr) {
	return arr.reduce((pre, cur) => {
  	return pre.concat(Array.isArray(cur) ? flat(cur) : cur)
  }, [])
}

// while 
// while 
function flat (arr) {
	while (arr.some(Array.isArray)) {
  	arr = [].concat(...arr)
  }
  return arr
}

```

#### 一维数组转二维数组
```javascript
function reflat (arr, len) {
  let result = []

  let loopNums = Math.ceil(arr.length/len)

  for (let i = 0; i < loopNums; i++) {
    result.push(arr.slice(i*len, (i+1) * len))
  }

  return(result)
}
```

####  js实现repeat方法 const repeatFunc = repeat(alert, 4, 3000); 调用这个 repeatedFunc("hellworld")，会alert4次 helloworld, 每次间隔3秒
```javascript
  function repeat(func, num, time) {
  var flag = 1;
  return function timer(str) {
    func(str)
    setTimeout(() => {
      flag++
      if (flag <= num) {
        timer(str)
      }
    }, time);
  }
}
```

#### script的async有什么用
正常情况下，script没有任何引入属性情况下，有以下特性
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/caf2f618530046658ab8e3b4a8699589~tplv-k3u1fbpfcp-zoom-1.image">
- js脚本分为加载(Fetch)、解析（Execution）、执行(Execution)
- js脚本的加载。解析、执行会阻塞DOM树渲染，因此一般会放置在尾部

defer 和 async有下面异同
- 都是属于异步加载脚本
- **defer加载完成后，到DOM解析完成后才会执行，但是会发生在documentContentLoaded之前**
##### async 
对脚本的请求是异步，不会阻塞html解析，一旦网络请求结果出来了，hmtl就会暂停解析，让js解析并执行相关代码
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/021b5dbeddb64db0a7099dc0a4dd076d~tplv-k3u1fbpfcp-zoom-1.image">

如果在async 请求发出来之后，html解析已经完成，那么对async不产生任何影响
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4e5a89a4a1fe49ed9d5acaf25ef9aadd~tplv-k3u1fbpfcp-zoom-1.image">


##### defer
脚本异步加载，获取脚本的请求的异步的，不会阻塞浏览器解析hmtl,一旦请求返回结果后，如果html解析还未完成，浏览器并不会解析并执行该返回结果脚本js代码，而是等到浏览器完全把hmtl解析完成在执行js代码     
**如果存在多个多个defer标签，会按照引入顺序解析执行，这样并不会影响相互执行U依赖关系**
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8313e4787f04c79838fec9961bda0fb~tplv-k3u1fbpfcp-zoom-1.image">

##### 使用场景
- 如果脚本是模块化，不存在依赖关系，那么请使用async
- 如果脚本依赖另一个脚本，那么请使用defer
- 如果脚本是很小，并且异步脚本依赖他，那么请使用一个没有任何属性的内联脚本

#### 手写正则表达式判断电话号码
```javascript
// 手机号码: 匹配1开头，第二位树是3-9的电话号码
const reg = /^1[3-9]\d{9}$/ig; 

// 座机号码 例如: (0511-4405222、021-87888822)
const reg = /\d{3}-\d{8}|\d{4}-\d{7}/ig; 
```

#### js转千分位
```javascript
  // Object.prototype.toLocaleString
  const number = 123456789;
  console.log(number.toLocaleString()) // 123,456,789


  // slice() 方法可从已有的数组中返回选定的元素,截取数组的一个方法
  function toThousandsNum(num) {
    var num = (num || 0).toString(),
      result = '';

    while (num.length > 3) {
      //此处用数组的slice方法，如果是负数，那么它规定从数组尾部开始算起的位置
      result = ',' + num.slice(-3) + result;
      num = num.slice(0, num.length - 3);
    }
    // 如果数字的开头为0,不需要逗号
    if (num) {
      result = num + result;
    }
    return result;
  }

  console.log(toThousandsNum(123456789123)); // 123,456,789,123

  // 数组从尾部遍历
  function toThousandsNum(num) {
    let count = 1;
    let result = '';
    let price_arr = price.toString().split('');

    for (let i = price_arr.length - 1; i >= 0; i--) {
      result = `${price_arr[i]}${result}`;
      if (count % 3 === 0 && i !== 0) {
        result = `,${result}`;
      }

      count++;
    }

    return result;
  }

  // reverse转化
  return price
  .toString()
  .split('')
  .reverse()
  .reduce((prev, cur, index) => {
    return index % 3 === 0 ? `${cur},${prev}` : `${cur}${prev}`;
  });

  console.log(toThousandsNum(123456789123)); // 123,456,789,123
```

#### sum(1)(2)(3).valueof()
```javascript
  function add() {
    let args = [...arguments];
    function _sum() {
      args.push(...arguments);
      return _sum;
    }

    // valueOf和toString，哪个先被改写优先调用谁，同时出现，调用valueOf
    _sum.toString = function () {
      return args.reduce((prev, cur) => prev + cur, 0);
    };
    return _sum;
  }
```

```javascript
  function sum() {
    const args = [...arguments];
    function sum_backup() {
      args.push(...arguments);
      return sum_backup;
    }

    sum_backup.valueof = () => {
      return args.reduce((prev, cur) => prev + cur);
    };

    return sum_backup;
  }

  console.log(sum(1)(2)(3)(4).valueof());
```

#### setTimeout一定会按时执行吗？
不会，setTimeout是宏任务，得等到同步任务、事件队列中微任务、排序靠前的宏任务执行完了后，才能执行当前宏任务事件
> setTimeout是一个异步的宏任务，当执行setTimeout时是将回调函数在指定的时间之后放入到宏任务队列。但如果此时主线程有很多同步代码在等待执行，或者微任务队列以及当前宏任务队列之前还有很多任务在排队等待执行，那么要等他们执行完成之后setTimeout的回调函数才会被执行，因此并不能保证在setTimeout中指定的时间立刻执行回调函数
```javascript
  setTimeout(function() {
    console.log("计时器执行") // 几秒之后才执行
  }, 0)


  for (var i = 0; i < 1000000000; i++) {
    if (i == 999999999) {
      console.log(i);
    }
  }
```

#### hash路由和history路由的区别
- hash模式是通过改变锚点(#)来更新页面URL，并不会触发页面重新加载，我们可以通过window.onhashchange监听到hash的改变。 仅仅改变hash，不会改变页面路径，刷新页面无影响
- history模式是通过调用window.history对象上的一系列方法来实现页面的无刷新跳转。改变了pathname,刷新页面会以当前路径请求服务器，可能会出现404情况

##### hash
- 可以改变URL，但不会触发页面重新加载（hash的改变会记录在window.hisotry中）因此并不算是一次http请求，所以这种模式不利于SEO优化
- 只能修改#后面的部分，因此只能跳转与当前URL同文档的URL
- 只能通过字符串改变URL
- 通过window.onhashchange监听hash的改变，借此实现无刷新跳转的功能

```javascript
  window.addEventListener('hashchange', () => {console.log(window.location.hash)})
```

##### history 
- 新的URL可以是与当前URL同源的任意 URL，也可以与当前URL一样，但是这样会把重复的一次操作记录到栈中
- 通过参数stateObject可以添加任意类型的数据到记录中
- 可额外设置title属性供后续使用
- 通过pushState、replaceState实现无刷新跳转的功能
- 需要服务端支持，不然一旦改变history， 刷新页面会导致404

window提供popState来监听history change
**可以监听到**
- 用户点击浏览器的前进和后退操作
- 手动调用 history 的 back、forward 和 go 方法

**<font color="red">监听不到</font>**
- window.history.pushState()
- window.history.replaceState()

```javascript
  window.history.pushState(initialObject, title, url)
  window.history.replaceState(initialObject, title, url);

  
  window.history.go(-1)
  window.history.back();
  window.history.forword() // 前进到下一个路由
```

##### history 用nginx部署
如果改变了history pathname。刷新页面，出导致nginx出现404，nginx以当前路径请求资源匹配不到时候， 需重定向到 /（index.html） 页面

##### 前端路由实现原理（history）
[深入理解前端中的 hash 和 history 路由](https://zhuanlan.zhihu.com/p/130995492)

#### 给定数组，求数组元素相加符合等于数字之和的组合 
```javascript
var combinationSum = function(candidates, target) {
    let res=[]
    function add(arr,sum,index){
        let num=candidates[index]
        arr=arr.concat(num)
        sum+=num
        if(sum>target){
            //失败
        }else if(sum===target){
            //成功
            res.push(arr)
        }else{
            for(let i=index;i<candidates.length;i++){
                add(arr,sum,i)
            }
        }
    }
    for(let i=0;i<candidates.length;i++){
        add([],0,i)
    }
    return res
};

```