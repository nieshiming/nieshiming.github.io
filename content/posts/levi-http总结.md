+++ 
draft = true
date = 2021-12-16T13:26:42+08:00
title = "levi_http总结"
description = ""
slug = ""
authors = []
tags = ["http", "八股文"]
categories = ["http"]
externalLink = ""
series = []
+++


#### http 协商缓存 & 强制缓存
- [http面试必会的：强制缓存和协商缓存](https://juejin.cn/post/6844903838768431118)
- [10分钟彻底搞懂Http的强制缓存和协商缓存](https://segmentfault.com/a/1190000016199807)

> 强缓存是利用http头部中的cache-control和expries控制

expries: (http1.0) 属于过时的验证缓存的方式，其值保存的是服务器返回的过期时间，再次发起请求的时候，如果客户端时间小于资源expires则不会发起请求      
cache-control: (http1.1) 现在最多使用的控制缓存的方式，服务器返回的一个相对时间(会设置一个毫秒数，在有效时间内发起服务器请求，会读取缓存内容，直至失效，重新发起服务器请求)，解决expires比较时间造成的问题，请求服务器资源时，比较缓存资源时间和客户端当前时间(发生更改)，则会重新请求资源

通常来说，强缓存不会向浏览器发起请求，直接从缓存中读取内容，在在chrome控制台的network的size选项中可以看到该请求返回200的状态码。分为 from disk cache 和 from memory cache。
- from disk cache：（硬盘中缓存） 一般非脚本内容，例如css/html
- from memory cache: （内存中缓存）一般是脚本、字体、图片

<!--more-->

**浏览器读取缓存优先级： memory > disk > 服务器请求**

浏览器缓存过程
- 浏览器第一次加载资源，服务器返回200，浏览器将资源文件从服务器上请求下载下来，并把response header及该请求的返回时间一并缓存
- 下一次加载资源时，先比较当前时间和上一次返回200时的时间差，如果没有超过cache-control设置的max-age，则没有过期，命中强缓存，不发请求直接从本地缓存读取该文件（如果浏览器不支持HTTP1.1，则用expires判断是否过期）；如果时间过期，则向服务器发送header带有If-None-Match和If-Modified-Since的请求
- 服务器收到请求后，优先根据Etag的值判断被请求的文件有没有做修改，Etag值一致则没有修改，命中协商缓存，返回304；如果不一致则有改动，直接返回新的资源文件带上新的Etag值并返回200
- 如果服务器收到的请求没有Etag值，则将If-Modified-Since和被请求文件的最后修改时间做比对，一致则命中协商缓存，返回304；不一致则返回新的last-modified和文件并返回200

> 强缓存情况下， 设置 cache-control 为 no-cache会走 协商缓存的判断，优先判断ETag , 没有的话判断last-modified对比和服务器资源修改时间，一直的话，返回200， 从缓存中读取  
> **ETag > last-modify**

<p>第一次访问资源</p>
<div style="text-align:center">
    <img  src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/851397db41d14ce1b9ce55ec75d59b86~tplv-k3u1fbpfcp-zoom-1.image" />
</div>

<p>第二次访问资源</p>
<div style="text-align:center">
    <img  src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b05a37c143634702b9e88d0a7143d0f1~tplv-k3u1fbpfcp-zoom-1.image" />
</div>

#### 浏览器输入URL地址后发生了什么
- 浏览器想DNS服务器查找输入URL对应的IP地址
- DNS服务器返回请求域名对应的ip地址
- 浏览器根据IP地址与目标web服务器建立起TCP连接
- 浏览器获取服务器返回的内容
- 浏览器渲染html、加载script脚本等
- 窗口关闭后浏览器终止与web服务器连接

参考文章
> **TCP三次握手  重点看文章**   
[三次握手、四次挥手告别](https://juejin.cn/post/6965544021833809928#comment)
>
> <img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/1f7570dc70d04876bb171cfdb973c8f6~tplv-k3u1fbpfcp-zoom-1.image">


那么为什么要三次握手呢？两次不行吗？
- 为了防止服务器端开启一些无用的连接增加服务器开销
- 防止已失效的连接请求报文段突然又传送到了服务端，因而产生错误


#### 常见http状态码
- 1xx, 表示请求已被接收，继续处理
    - 100 客户端必须继续发出请求
    - 101 客户端要求服务端根据请求转换http协议版本
- 2xx, 成功–表示请求已被成功接收、理解、接受
    - 200 请求成功
- 3xx, 重定向，完成请求必须更进一步的操作
    - 301 （永久移动） 请求的网页已永久移动到新位置。 服务器返回此响应（对 GET 或 HEAD 请求的响应）时，会自动将请求者转到新位置
- 4xx：客户端错误–请求有语法错误或请求无法实现。   
    - 400 （错误请求） 服务器不理解请求的语法
    - 401 （未授权） 请求要求身份验证。 对于需要登录的网页，服务器可能返回此响应
    - 403  服务器拒绝响应
    - 404  请求地址找不到
- 5xx：服务器端错误–服务器未能实现合法的请求
    - 500 （服务器内部错误） 服务器遇到错误，无法完成请求
    - 502 （错误网关） 服务器作为网关或代理，从上游服务器收到无效响应
    - 504 （网关超时） 服务器作为网关或代理，但是没有及时从上游服务器收到请求
    - 505 （HTTP 版本不受支持） 服务器不支持请求中所用的 HTTP 协议版本

#### 请列举常用的 HTTP 方法，并介绍 GET 与 POST 请求之间的区别
- get请求数据量较小，可以在URL展示出来,可以被缓存
- post请求，报文数据量比get多

#### cookies sessionstorage和localstorage的区别
- cookie：存储在用户本地终端上的数据
- cookie数据始终在同源的http请求中携带(cookie在浏览器和服务器间来回传递)， 每次http请求都会携带cookie 大小为4k
- sessionstorage和localstorage存储在浏览器中，不参与服务器的通信,大于5M
- sessionstorage：客户端存储数据，当用户关闭浏览器以后，数据就会被删除
- localstorage：客户端存储数据,没有时间限制的数据存储


#### HTTP 和 HTTPS 的共同点和区别
经典五层模型： 应用层(http/https)、传输层(TCP)、网络层(IP)、数据链路层  
http: 超文本传输协议    
https： 超文本传输安全协议    
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/461ea67a5bb94ac9aa68708742efea9f~tplv-k3u1fbpfcp-zoom-1.image">
- https 协议需要到 ca 申请证书，一般免费证书较少，因而需要一定费用
- http 和 https 使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443
- http 是超文本传输协议，信息是明文传输， https 则是具有安全性的TLS/SSHL加密传输协议，相当加了一层(安全层)， 更加消费服务器资源
- http请求比https响应更快， 因为https加了TLS，建立连接更加复杂，也要交换更多的数据，难免会影响速度
- http的连接很简单，是无状态的，https协议是有SSL+HTTP 协议构建的可进行加密传输，身份认证的网络协议，比http更加安全

#### HTTPS加密过程

##### 对称加密
> 简单来说就是一个密钥，可以用它加密数据，也可以用它解密数据, eg: 客户端和服务器用同一个密钥来进行加密、解密工作
缺点：密钥容易受到第三方劫持
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/80810561f8eb4db281bea525828c2fab~tplv-k3u1fbpfcp-zoom-1.image">


##### 非对称加密
> 简单来说就是有2个密钥： 公钥和私钥。若使用私钥加密，公钥可用来解密。反之，可以用公钥加密，私钥解密
eg: 服务器维护一组公钥、私钥。在客户端TLS阶段传递客户端公钥，客户端用公钥对数据解密。 服务器解密操作。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/87d30e822b4241d8b685276600a8304e~tplv-k3u1fbpfcp-zoom-1.image">

##### 握手流程
- 客户端通过URL访问服务器建立TLS连接
- 服务器把证书、公钥、公司信息，证书有效期等信息传递给客户端
- 客户端随机生成一个对称密钥(随机key)，在用公钥加密尬该密钥，加密后密钥回传给服务器
- 服务器用私钥解密客户端传递密钥
- ...后续客户端和服务端就用该密钥进行数据通信

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f98f6bceeffc441caa5eb250ff5098c5~tplv-k3u1fbpfcp-zoom-1.image">


##### 每次进行HTTPS请求时都必须在SSL/TLS层进行握手传输密钥吗？
服务器会给每个客户端维护一个sessionId,在TLS阶段传递给浏览器，浏览器生成好的密钥传递给服务器，服务器会把密钥和sessionId对应存在起来，之后再浏览器每次请求的时候
带上sessionId，服务器根据sessionId,找到对应的密钥并进行解密工作。这样就不用每次让客户端只做密钥，服务器保存密钥操作了。     

参考文章：
- [彻底搞懂HTTPS的加密原理 - 知乎](https://zhuanlan.zhihu.com/p/43789231)
- [HTTPS 详解一：附带最精美详尽的 HTTPS 原理图](https://segmentfault.com/a/1190000021494676)

#### 什么是跨域
跨域本质是浏览器的同源策略造成，指的是浏览器不能执行其他网站的脚本
> 什么是同源策略： 协议，端口，域名
- 同源策略限制了一下行为
 - Cookie、LocalStorage 和 IndexDB 无法读取
 - Ajax请求可以发出来，但是接收不到响应

#####  解决跨域
- 利用jsonp跨域，  传递服务器callback函数，同时callback挂载在window上 => 缺点只支持get，存在csrf风险
```javascript
// 客户端代码
const Jsonp = async (
  request: { params: object; callback: string; url: string },
  timeout: number = 2000
) => {
  return new Promise((resolve, reject) => {
    const { url, params, callback } = request;
    const method = `${callback}_${Date.now()}`;

    let src: string = url.indexOf('?') > -1 ? url : `${url}?`;
    for (let key in params) {
      src += `&${key}=${params[key]}`;
    }

    const script = document.createElement('script');
    script.setAttribute('src', `${src}&callback=${encodeURIComponent(method)}`);
    document.body.appendChild(script);

    const clean = () => {
      clearTimeout(timer);
      delete window[method];
      document.body.removeChild(script);
    };

    const timer = setTimeout(() => {
      clean();
      reject('timeout 超时啦');
    }, timeout);

    window[method] = data => {
      clean();
      resolve(data);
    };

    script.addEventListener('error', () => {
      clean();
      reject('资源加载失败');
    });
  });
};

// 服务端
  const qs = require('qs')
  const http = require('http')

  http
    .createServer((req, res) => {
      const [path, query] = req.url.split('?')
      const params = qs.parse(query, { ignoreQueryPrefix: true })
      res.writeHead(200, { 'Content-Type': 'text/javascript' })
      setTimeout(() => {
        res.end(
          `${params.callback}(${JSON.stringify({
            path,
            name: params.name,
            date: params.date,
          })})`
        )
      }, 3000)
    })
    .listen(8081)

  console.log('http serve running at 8081')
```

- window.name
- postMessage 跨域
  > 实现： window.top.postMessage(message, targetOrigin);  message: 需要传递数据， targetOrigin: 通过origin指定哪些窗口可以接受消息事件(message, 可以设置为 * )
  新老crm模块切换使用，老的crm任务单还未下线，新的已经完成
``` javascript
  // 新crm逻辑， 部分页面跳转到指定老的crm详情页
  //  searchresult组件 判断是否跳转新老详情页面
  private gotoDetail = (item: ITask) => {
    let url = `/${ENV_CONFIG.baseName}/task/detail/${item.taskKey}`;

    if (getQueryString('viewtype') === 'old' || this.props.isIframe) {
      switch (item.taskCategoryCode) {
        case ETaskCategory.审批:
        case ETaskCategory.升级处理:
        case ETaskCategory.拍拍升级处理:
        case ETaskCategory.新机退款:
          url = `/${ENV_CONFIG.baseName}/task/detail/${item.taskKey}`;
          break;
        default:
          url = `/#/mission/${item.taskKey}`;
      }
    }

    if (this.props.isIframe) {
      window.top['postMessage']({ type: 'OPEN_WINDOW', params: url.replace(/\/#\//, '/') }, '*');
      return;
    }

    window.open(url);
  };

  //  老crm，接受postMessage信息，判断路由跳转， 老crm有嵌套iframe部分
  <iframe
    v-if="this.conversation.phone || this.orderId"
    border="0"
    frameborder="no"
    marginwidth="0"
      marginheight="0"
    :src="'/lordaeron/task/searchresult?userMobile=' + this.conversation.phone + '&busiKey=' + this.orderId"
    :style="{ width: '100%', height: iframeHeight }"
  ></iframe>

  window.addEventListener("message", (d) => {
    let { data } = d;
    let { type, params } = data || {};
    if (type === "OPEN_WINDOW") {
      if (/lordaeron/.test(params)) {
        window.open(params);
        return;
      }
      this.routeTo({
        path: params,
        type: "_blank",
      });
    }

    if (type === "PHONE_CALLING") {
      document.querySelector('#dialout_input').value = params;
      document.querySelector('#DialEnable').click();
    }
  });
```


- 跨域资源共享CORS --- 不支持IE9以下浏览器
- nginx 实现反向代理， webpackdevserver proxy 是一个小型服务器，也能实现反向代理指定到指定域名


#### 跨域资源共享CORS（简单请求、复杂请求）
跨站资源共享中的一种方式，它使用额外的http头部告诉浏览器可以服务器进行跨域资源请求
##### 请求类型
###### 简单请求
- 请求方式是 GET, POST, HEAD一种
  HEAD请求：和GET请求本质没说什么区别，也是从服务器获取资源，服务器处理逻辑也是一样的。只不过服务器不糊返回请求内容实体。只会回传响应头。
  1、常用来判断资源是否存在， 2、检查文件是否最新版本
- 无定义头部信息
- content-type值只能是下面几种
  - application/x-www-form-urlencoded
  - multipart/form-data
  - text/plain

###### 复杂请求
当一个请求不满足上诉简单请求，被判定为复杂请求
复杂请求有2个步骤，开始是options请求（预检请求），然后是实际请求，    
> CORS预检请求发生在实际请求之前，用于验证服务器是否支持复杂请求，是否安全等、避免跨域请求对用户数据产生未预期结果，在实际业务中经常有自定义头部存在，
这种自定义类型无法预测和保证数据安全，所以需要一个协商的过程(预检请求)



##### CORS请求过程
CORS请求需要要求服务器设置允许访问来源，即：设置**Access-Conrol-Allow-Origin**，该字段表明服务器允许哪些站点可以跨域访问资源， 

- Origin: 请求发出的原站，如果不在Access-Control-Allow-Origin范围内，则拒绝请求
- Access-Control-Allow-Origin: * ; 代表允许所有站点访问， 注意如果设置 * ，那么头部设置 withCredentials： true， ，无法携带cookie
- Access-control-Allow-Methods: 约定的请求方法（GET/PUT/DELETE/POST/PATCH等）
- Access-Control-Allow-Header: 自定义约束的头部
- Access-Control-Expose-Headers: 跨域请求中，浏览器默认情况下通过API只能获取到以下响应头部字段：
  - cache-control
  - last-modify
  - expries
  - content-type
- Access-Control-Allow-Credentials: boolean;  表示在跨域请求是否允许客户端发送cookie给服务器。
**浏览器cors请求默认不发生cookie，需要传递cookie的话，客户端和服务端同步设置 withCredentials： true  和 Access-Control-Allow-Credentials: true, 并且Access-Control-Allow-Origin: 不是*， 不然会报错**

<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a70473688cc143d48870eca4c5e627f7~tplv-k3u1fbpfcp-zoom-1.image">

#### http1.0 / http1.1 / http2.0 区别
- http1.1 增加了connection: keep-alive(保持http持久连接)
   > 假设有6个http请求，如果开启keep-alive, 那么在5-15s同时发送6次http请求，走的是一个TCP连接，走一个是同一条http流，如果超出5-15没有发生http请求，那么连接中断，这表示是一条http长连接  
    如果不使用http持久连接，那么没发生一次http请求，都会经历TCP三次握手，非常耗时    
    **优点：降低服务器因大量建立TCP建立而造成CPU加重负载，以及TCP传输相关阻塞问题**     
    **缺点：以前带宽小，瞬时请求高，所以用这个方法降低 TCP 新建。但现在带宽大，并发高。如果 HTTP 服务存在长轮训或较长间隔请求，而且超过 Keep-Alive 的设置（比如 Keep-Alive 5 秒，但轮训周期是 6 秒），则可能会造成大量的无用途连接，白白占用系统资源 ==> 浪费带宽资源**  
- http 1.1 增加host字段
- http 2.0 
    - 头部压缩： 传递头信息之前会进行压缩
    - 二进制分帧：头部信息使用二进制的格式并分成一帧一帧传递 (1.1是一个文本格式)
    - 多路复用： 1.1请求和响应是有顺序，一旦某个前面未响应后续请求会阻塞，2.0响应是无序的，不会造成阻塞
    - 服务器推送： 服务器可以对客户端一次请求响应多次，例如客户端请求html，服务器可以额外推送css/js/img资源等

#### web常用的攻击技术和防护
> xss 跨站脚本攻击: XSS 攻击是指攻击者在网站上注入恶意的客户端代码，通过恶意脚本对客户端网页进行篡改，从而在用户浏览网页时，对用户浏览器进行控制或者获取用户隐私数据的一种攻击方式

> csrf 跨站请求伪造: 通常情况下，CSRF 攻击是攻击者借助受害者的 Cookie 骗取服务器的信任，可以在受害者毫不知情的情况下以受害者名义伪造请求发送给受攻击服务器，从而在并未授权的情况下执行在权限保护之下的操作。

- [浅说 XSS 和 CSRF](https://github.com/dwqs/blog/issues/68)
- [浏览器专题之安全篇](https://bubuzou.com/2020/12/04/web-security/)

参考文章
- [15个常见的http知识点复习](https://juejin.cn/post/6844903872935247886)
- [connection keep-alive作用](https://segmentfault.com/q/1010000019503002)

#### DNS解析相关 && CDN原理
CDN中文含义”内容分发网络“，将原站的内容分布到接近用户边缘的边缘。用户可以就近获取数据
- 降低了网络请求阻塞情况
- 提高了请求的响应的数据
- 减少了原站的负载压力

##### 访问原站的过程(无CDN)

- 用户在客户端输入相关域名。 ”https://abc.sina.com“;
- 客户端首先会在本地hosts或者hosts缓存中查到域名对应ip地址;
- 如果没有信息，则会到本地DNS查询域名对应服务器地址;
- 本地DNS仍然没有域名对应的ip地址，则会依次 根DNS、顶级域名DNS进行询问。最终本地DNS把域名对应ip地址返回给客户端;
- 客户端拿到对应ip地址，经过标准的TCP三次握手、建立TCP连接； 浏览器想服务器发起数据请求 =>  服务器把数据返回给客户端
- 客户端解析数据（html、css、js、image等资源）
- 经过标准的TCP挥手流程，断开TCP连接

##### 域名解析
域名解析可以分为2种
- 将一个域名解析成一个ip地址
- 将一个域名解析成另一个域名

其实解析思路不难，我们在域名服务商购买了一个域名之后，需要去映射一个IP地址，可以用Map来表示这个关系：{域名：IP}。同时我们也可以给某个域名取一个别名，比如“www.baidu.com”取一个别名“test.baidu.com”，这种关系也可以用Map来表示：{域名：别名}。这里的别名专业一点叫做CNAME，相信大家对这个词有点眼熟，它就是这个意思
**而域名解析： 最终解析为一个ip地址或者一个CNAME**

域名解析是由DNS完成，如果发现域名解析为CNAME,那么继续查询CNAME对应的ip地址


##### 使用CDN获取缓存内容过程
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/42a062e642ce473ab2e1bd0e76f91b29~tplv-k3u1fbpfcp-zoom-1.image">

> 用户使用某个域名来访问静态资源时（这个域名在阿里CDN服务中叫做“加速域名”），比如这个域名为“image.baidu.com”，它对应一个CNAME，叫做“cdn.ali.com”，那么普通DNS服务器（区别CDN专用DNS服务器）在解析“image.baidu.com”时，会先解析成“cdn.ali.com”，普通DNS服务器发现该域名对应的也是一个DNS服务器，那么会将域名解析工作转交给该DNS服务器，该DNS服务器就是CDN专用DNS服务器。CDN专用DNS服务器对“cdn.ali.com”进行解析，然后依据服务器上记录的所有CDN

- 客户端发起域名请求，先经过本机hosts、hosts缓存查看对应的ip，如果本机没有对应ip地址，转向本地DNS解析、域名DNS、顶级域名DNS依次查找，如果发现该域名对应一个CNAME(CND别名域名)。则会把域名解析权交给CDN专属DNS服务器
- CDN专属DNS服务器把CDN全局负载均衡设备(GSLB)ip返回给客户端
- 用户向全局负载均衡设备ip发起网络请求
- GSLB根据请求ip判断用户当前位置。筛选出该区域内最优的区域负载均衡设备(SLB),并把请求转发该设备上
- 区域负载均衡设备(SLB)考虑每个节点的负载情况、健康度、、连接数资源限制条件。筛选出最优缓存节点,并将http请求最终转发到该节点上
- 用户向获取的IP地址发起对该资源的访问请求。
  - 如果ip地址对应节点已经缓存了资源，则会直接把数据返给用户
  - 如果该ip地址对应节点未缓存该资源。则会向源站请求内容，结合用户自定义配置的缓存策略，把内容缓存到改节点上，并返回给用户，请求结束

##### 参考文章
- [CDN原理简析](https://juejin.cn/post/6844903873518239752)
- [漫话：如何给女朋友解释什么是CDN？](https://juejin.cn/post/6844903906296725518)
- [百度面试官：给我说是CDN加速的原理？](https://zhuanlan.zhihu.com/p/306069528)


#### 使用Promise封装ajax请求
```javascript
    /***
     * @description
     *
     * readyState: 0 请求还未建立，未执行open方法
     * readyState: 1 请求已建立，还未调用send方法
     * readyState: 2 send方法执行
     * readyState: 3 处于响应中，还未得到response
     * readyState: 4 响应结束，请求完成处理完成
     * */
    const levi_ajax = (url, method, params, headers) => {
      return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open(method, url, true);  //  xhrReq.open(method, url, async);  async 是否异步，默认true

        for (let key in headers) {
          let value = headers(key);
          xhr.setRequestHeader(key, value);
        }

        xhr.onreadystatechange = () => {
          if (xhr.readyState === 4) {
            if (xhr.status === 200) {
              resolve(xhr.responseText);
            } else {
              reject('some errors');
            }
          }
        };

        xhr.send(params);
      });
    };

    // 改进，使用onload、onerror监听
    const levi_ajaxb = (method, url, params, headers) => {
      return new Promise((resolve, reject) => {
        const xhr = new XMLHttpRequest();
        xhr.open(method, url);

        for (let key in headers) {
          xhr.setRequestHeader(key, headers[key]);
        }

        xhr.onload = () => {
          if (xhr.readyState === 4) {
            if (xhr.status === 20) {
              resolve(xhr.responseText);
            } else {
              reject(xhr.statusText);
            }
          }
        };

        xhr.onerror = () => {
          reject('some error');
        };

        xhr.send(params);
      });
    };
```

#### content-type 有几种 **
用于定义网络文件以及类型，同时告知浏览器以那些方式处理该文件
- 文本： text/html、text/javascript、text/css 、 text/plain
- 图片： image/gif、image/jpeg、image/png
- 视频： video/ogg
- 视频： audio/ogg 、 audio/wav
- 二进制：application/json 、 application/pdf
- 上传文件： multipart/form-data

#### http报文结构，有哪些headers **
用户http协议交互的信息成为报文： 有请求报文，响应报文， http报文可分为报文首部、报文主体
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/033d324cd6ac4378910f3db095f7a66c~tplv-k3u1fbpfcp-zoom-1.image">

##### 请求报文
- 请求行（请求方法、请求url、协议版本）
- 请求头部
- 空行
- 请求体： 要传递的参数

**实例数据**
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/00ca77a9358048359f2c5b90d8fc4bd2~tplv-k3u1fbpfcp-zoom-1.image">

**Request Header**
- referer
- cookie
- origin
- accpet
- accept-language
- content-type
- cache-control
- xxx自定义参数


##### 响应报文
- 状态行
- 响应头部
- 空格
- 响应体

**实例数据**
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bafeb9a8323a4ca4add4a6be3f99f1a9~tplv-k3u1fbpfcp-zoom-1.image">

**Response Header**
- ETag
- date
- content-type
- last-modified
- ...自定义参数


##### 参考文章
- [一篇让你彻底了解http请求报文和响应报文的结构](https://juejin.cn/post/6920784699493187591)

#### 你一般用的MIME类型有哪些？**
通常来说，浏览器通过MIME Type区分不同的媒体资源， 在把输出结果响应到浏览器上的时候，浏览器必须启动适当的应用程序来处理这个输出文档。这可以通过MIME来完成。在HTTP中，MIME类型被定义在Content-Type header中     
**mime类型：参考上文：content-type类型**
