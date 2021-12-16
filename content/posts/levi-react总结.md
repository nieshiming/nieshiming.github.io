+++ 
draft = true
date = 2021-12-16T13:27:27+08:00
title = "levi-react总结"
description = ""
slug = ""
authors = []
tags = ["react", "八股文"]
categories = ["react"]
externalLink = ""
series = []
+++


#### 什么是虚拟DOM，怎么构成的
虚拟DOM是真实DOM在内存中的表示，简单来说，虚拟DOM就是个对象，由tag,props,children构成
```javascript
  <div id="app">
    <p class="text">hello world!!!</p>
  </div>

  // 可转化成下面的虚拟DOM表示
  {
    tag: 'div',
    props: {
      id: 'app'
    },
    chidren: [
      {
        tag: 'p',
        props: {
          className: 'text'
        },
        chidren: [
          'hello world!!!'
        ]
      }
    ]
  }
```
上面对象就是我们说的虚拟DOM，可表示成树形结构，原生 DOM 因为浏览器厂商需要实现众多的规范（各种 HTML5 属性、DOM事件），即使创建一个空的 div 也要付出昂贵的代价。虚拟 DOM 提升性能的点在于 DOM 发生变化的时候，通过 diff 算法比对 JavaScript 原生对象，计算出需要变更的 DOM，然后只对变化的 DOM 进行操作，而不是更新整个视图

<!--more-->

##### react
主流的虚拟DOM库都有一个h函数，用于降虚拟DOM，转化为真实DOM, react通过babel把jsx转换成h函数形式，即： react_createElement函数， 最后调用render函数将虚拟DOM插入htmlDOM树中，渲染页面。 
示例代码通过babel编译jsx转换成react可处理的结构
```javascript

  const Page = () => {
    return <div 
            className="levi"
            style={{margin: '100px'}} 
            onClick={onClick}
            >hello world</div>
    }

  export default Page

  // babel编译后
  const Page = () => {
    return /*#__PURE__*/_react.default.createElement("div", {
      className: "levi",
      style: {
        margin: '100px'
      },
      onClick: onClick
    }, "hello world");
  };

  var _default = Page;
```

#### 谈谈你对DIFF算法理解
顾名思义：就是比较新老VDOM变化，把变化的部分更新到视图上  
**diff的优化策略**
- 针对于tree diff优化，react会忽略DOM节点跨层级的移动操作
- 针对于组件 diff优化，如果是不同的组件就会生成不同的组件，所以如果是不同的组件，就会被标记要修改，而不再细致的比较
- 针对于同一层级的一组子节点，可以通过对应的key进行区别

##### tree diff
针对于dom跨层级移动操作，diff算法会对相同层级的节点比较，如果某个节点不存在了，那么这个节点和其子节点是不会再有比较的，react只会考虑同层级的节点的位置变化，而对于不同层级的节点，只有创建和删除。如果某个节点下发现某个子节点A消失了，就会直接销毁子节点A。
<img  src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b64d5beb43b241ecaac13e3d6eeffde2~tplv-k3u1fbpfcp-watermark.image">

##### component diff
如果是同一类型的组件，则会比较虚拟dom树，如果不是同一类型，那么这个组件将被标记为dirty组件，从而替换整个组件下的所有子节点。如果是同一类型的组件，可能存在虚拟dom没有任何变化，因此react通过shouldComponentUpdate来判断这个组件是否需要进行diff运算, 如下图，一旦react判断D和G是不同类型的组件，就不会比较两者的结构，而是直接删除componentD，重新创建组件G及其子节点
<img src="https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9baecab178494a07bf5c480aac11f6e4~tplv-k3u1fbpfcp-watermark.image">

##### element diff
当节点处于同一级时，diff提供三种节点操作：插入，移动，删除, 如下图，老集合节点顺序为：a,b,c,d，新节点的顺序为：b,a,d,c。如果是传统的diff操作，那么就会对比老A和新B，发现B!=A于是创建并插入B至最新的集合中，删除老集合中的A，以此类推。
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c791b241d8b74c7498d8f069dc583bef~tplv-k3u1fbpfcp-watermark.image">
React提出，通过key来判断同一层级的同组子节点。如果是相同的节点，只需要移动位置即可
<img src="https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b8426edda46c4a16a691a81509aa5b44~tplv-k3u1fbpfcp-watermark.image">
<font color=red>react主要用key来区分组件，相同的key表示同一个组件，react不会重新销毁创建组件实例，只可能更新。如果key不同，react会销毁已有的组件实例，重新创建组件新的实例</font>

#### 生命周期有哪些
react > 16.3版本最新的生命周期,   
<img src="https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a5236362d1e842d99b30cd04d3633157~tplv-k3u1fbpfcp-watermark.image">

移除了3个生命周期
  - componentWillMount
  - componentWillReciveProps
  - componentWillUpdate

新增了2个生命周期
 - static getDerivedStateFromProps
 - getSnapshotBeforeUpdate

我们将react生命周期化为为三个阶段，分别是
- 挂载阶段 Mouting
- 更新阶段 Update
- 卸载阶段 unMount

##### 挂载阶段
组件初始化阶段，这个阶段只会执行一次，有以下生命周期方法，顺序如下
- constructor
- staic getDerivedStateFromProps
- render
- componentDidMount

> constructor 构造函数，初始化阶段被执行，如果使用了构造函数，需要在内部执行super(props)，不然会报错，
在构造函数常做以下2件事情
- 初始化state对象
- 自定义事件(合成事件)绑定this 
```javascript
  class Levi extends React.Component{
    constructor(props) {
      super(props);
      
      this.handleClick = this.handleClick.bind(this);
    }
  }

```

> getDerivedStateFromProps react实例的静态方法，<font color=red>不能在该方法中使用this</font>，接受2个参数： 即使 getDerivedStateFromProps(nextProps, prevState)
> 这个函数会返回一个对象用于更新当前state,如果返回null，则不更新state

```javascript
  class Levi extends React.Componet {
    state: {
      age: 20
    }

    static getDerivedStateFromProps(nextProps, prevState) {
      if (nextProps.currentRow !== prevState.lastRow) {
        return {
            lastRow: nextProps.currentRow
        }
      }

      return null
    }
  }
```

> render react最核心方法，一个组件中必须有这个方法，是一个纯函数，返回需要被渲染的元素，不要这个方法包含业务逻辑，例如数据请求，可能会造成重复渲染，页面卡死

> componentDidMount  组件装修后调用，常用于数据请求，获取并操作DOM。可以写一些订阅操作，但是记得在componentWillUnMount 取消订阅


##### 更新阶段
组件props、state发生改变，会触发该阶段

- getDevrvedStateFromProps
- shouldCompoentUpdate
- render
- getSnapShotBeforeUpdate
- componentDidUpdate

> getDevrvedStateFromProps 同上

> shouldCompnentUpdate 接受2个函数 为 shouldCompnentUpdate(nextProps, nextState)会返回一个boolean， true表示执行更新，页面渲染， false表示页面不会渲染, 默认返回true
<font color=red>注意当我们调用forceUpdate并不会触发此方法</font>。不建议手工编写shouldCompnentUpdate逻辑，除非牵扯到复杂的深度比较<shouldComponentUpdate默认浅层比较props,state, 可在方法内部自定义比较逻辑>，官方推荐使用 React.PureComponent<浅层比较props,state>

> render 同上

> getSnopShotBeforeUpdate 这个方法在render之后、componentDidUpdate之前，它使您的组件可以在可能更改之前从DOM捕获一些信息（例如滚动位置。 有2个参数： getSnapShotBeforeUpdate(prevProps, prevState), 函数会返回一个值，会作为低三个参数传递给componentDidUpdate. 如果没有返回值，请返回null，不然控制台会有警告，并且componentDidUpdate不能缺少，不然也会警告。 
```javascript

    // 限制聊天气泡框scrollTop: https://kinyaying.github.io/webpack-optimize/dist/#/snapshotSampleOptimize
  class Levi extends React.Componet {
    getSnapshotBeforeUpdate(prevProps, prevState) {
      state =  {
          newList: []
      }

      leviRef = React.crateRef();

      if (prevProps.list.length < this.state.newList.length) {
        return this.leviRef.scrollHeight;
      }

      return null;
    }

    componentDidUpdate(prevProps, prevState, perScrollHeight) {
      if(!snopShot) {
        console.log('这里是getSnapShotBeforeUpdate 拿到返回值')
        
        const curScrollTop= this.leviRef.scrollTop;
        if (curScrollTop < 5) return ;
        this.leviRef.scrollTop = curScrollTop + (this.rootNode.scrollHeight  - perScrollHeight);   //加上增加的div高度，就相当于不动
      }
    }
  }
```

> componentDidUpdate 发生在getShapShotBeforeUpdate之后，接受3个参数分为： componentUpdate(prevProps, prevState, snapshot)
> 在这个函数里面可以进行DOM操作和数据请求，但是请注意，如果setState。 一定要写if判断逻辑，不然会造成死循环，页面卡死

参考文章
- [React 17生命周期总结](https://zhuanlan.zhihu.com/p/370198189)

##### 卸载阶段
- componentWillUnmount
> componentWillUnmount 组件卸载时调用， 可取消订阅、清除定时器、清除组件内部引用store的数据等操作

#### setState
- setState 只在合成事件和钩子函数中是“异步”的，在原生事件和 setTimeout 中都是同步的
- setState的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果
- 在原生事件和setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次 setState ， setState 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 setState 多个不同的值，在更新时会对其进行合并批量更新

> setState 是通过 enqueueUpdate 来执行 state 更新的，那 enqueueUpdate 是如何实现更新 state 的？继续往下走。
3、enqueueUpdate 如果当前正处于创建/更新组件的过程，就不会立刻去更新组件，而是先把当前的组件放在 dirtyComponent 里，这里也很好的解释了上面的例子，不是每一次的 setState 都会更新组件。否则执行 batchedUpdates 进行批量更新组件；  

```javascript
enqueueSetState: function(publicInstance, partialState, ...) {
  var internalInstance = getInternalInstanceReadyForUpdate(publicInstance);
  if (internalInstance) {
    (internalInstance._pendingStateQueue || (internalInstance._pendingStateQueue = [])).push(partialState), 
    enqueueUpdate(internalInstance);
  }
}

function enqueueUpdate(component) {
 // ...  
 if (!batchingStrategy.isBatchingUpdates) {
    batchingStrategy.batchedUpdates(enqueueUpdate, component);
    return;
  }

  dirtyComponents.push(component);
}
```
setState的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果
<div>
    <img src="https://github.com/JTangming/tm/blob/master/images/react/setState.png?raw=true">
</div>

参考文章
- [setState异步、同步与进阶](https://juejin.cn/post/6844903715921477640#heading-2)
- [关于 React setState，你了解多少？ #11](https://github.com/JTangming/blog/issues/11)


#### 类组件和函数组件区别
类组件就是常说的class组件，有声明周期以及state    
函数式组件：无声明周期，可接受props，常用来作为UI组件，
> 函数式组件性能比类组件要高，适用于表现层，类组件包含声明周期以及state特性，可处理复杂的逻辑，适用于逻辑层。<font color=red>react初衷偏向于使用函数式组件，这样能更快的渲染。hooks横空出世</font>

#### react ref作用
dom元素、组件真正实例的引用，其实就是react.render()函数执行后，返回的**组件实例**， <font color=red>你不能在函数组件上使用 ref 属性，因为他们没有实例。</font>

- 为什么会用到refs
  当要处理DOM元素的时候，例如元素focus,文本的选择，媒体的播放，第三方dom库集成，特性情况获取元素的宽高度等，这些是react无法控制的局面，所以需要用到ref

- 如何创建ref

  ```javascript
  <!-- react createRef -->
  class Levi extends React.Component {
    constructor(props) {
      this.leviRef = React.createRef();
    }

    private init() {
      console.log(this.leviRef)
    }

    render() {
      return <div ref={this.leviRef}>hello world</div>;
    }
  }

  <!-- hooks -->
  const Page = () => {
    const leviRef = useRef(initialVal);

    const fetch = useCallback(() => {
      console.log(leviRef.current)
    }, [])

    return  <div ref={this.leviRef}>hello world</div>;
  }
  ```

- 访问ref
当ref传递给render元素时，対该元素的引用可在ref中的current获取到
> const ref = this.leviRef.current

#### 什么是高阶组件？
高阶组件可接受react组件作为参数并返回一个新的组件的函数，
高阶组件可用于以下场景
- 代码复用，逻辑抽象
- 渲染劫持
- props处理，
- state抽象、操作
- 可用过ref访问该组件的实例
```javascript

const HOCComponent = (WrapComponent) => {
  return class HOCComponent extends React.Component {
      state = {
        name: 'levi',
        age: 20
      }
      levi = React.createRef();

      componentDidMount() {
        console.log(this.levi.current); // 打印组件实例
        this.setState({age: 21}) // fecth data
      }

      render() {
        const newProps = {
          area: 'sh'
        }

        // 起到权限控制作用，不用再每个月页面里面判断
        if(this.state.age < 18) {
          return <div>未成年人无权限进入</div>
        }

        return <div><WrapComponent {...newProps} {...state} ref={this.levi} /><div>
      }
  }
}
```

#### 什么是jsx
- 优点、特点
  - 增强可读性
  - 语法上和HTML非常相似
  - 允许自定义标签以及组件
  - 抽象可读性，react进行升级，不需要改动任何jsx代码
  - 模块化，保证了组件中的标签与业务逻辑相分离


- jsx和html之间的不同
  - jsx可以使用变量属性
  - 事件要使用驼峰式写法
  - html使用的是class，而jsx使用的是className
  - style的css属性要使用驼峰式写法

- jsx 会经历什么，变成什么样子，怎么渲染到页面的
jsx会经过bable编译+react.js构造，变成js对象（这个对象也就是我们说的虚拟dom），然后通过ReactDOM.render进行创建DOM元素，然后将DOM插入到浏览器中

- jsx为什么要经历一层bable编译呢？
jsx是一个抽象出来的语法，它经过bable编译以后，可以变成一个可以描述UI的对象，我们可以拿着这个对象，渲染到浏览器，也可以选择显示到手机APP，所以这也是为什么要把react-dom单独抽离出来

#### 什么是React.Context
context的作用就是实现跨层级的组件数据传递， context提供了一种在组件之间可以共享值的方法，而无需通过树的每个级显示传递props    
**优点： 跨组件访问数据**
**缺点：react组件树中某个上级组件shouldComponentUpdate返回false以后，当context更新，是不会引起下级组件的更新**      


函数式组件React.useContext上层最近的 <MyContext.Provider> 更新时，该 Hook 会触发重渲染，并使用最新传递给 MyContext provider 的 context value 值。
即使祖先使用 React.memo 或 shouldComponentUpdate，也会在组件本身使用 useContext 时重新渲染

##### 如何创建
```javascript
  // 创建context上下文，可单独抽离成一个文件
  const LeviInfoContext = React.createContext({ name: 'levi', age: 20, run: () => console.log('run') });

  // 提供provider包含内组件消费, 接受一个默认值value
  class App extends React.Component {
    public render() {
      return (
        <LeviInfoContext.Provider value={{ name: 'levi', age: 21 }}>
          <App />
        </LeviInfoContext.Provider>
      );
    }
  }

  // 类组件使用consemer消费
  import LeviInfoContext from 'xxxx';
  class Children extends React.Component {
    public render() {
      return <LeviInfoContext.consumer>{(baseObject) => <div>i am child</div>}</LeviInfoContext.consumer>;
    }
  }

  // 函数式组件使用 React.useContext。接受React.createContext返回的值
  const Page = () => {
    const info = React.useContext(LeviInfoContext);

    return <div>11111{info.name}</div>;
  };
```

[从Context源码实现谈React性能优化](https://juejin.cn/post/6907546624441090055)

#### 如何避免组件重复渲染
React常见问题就是重复渲染，React提供了2个方法，避免重复渲染
- React.Memo(component, fn)
- React.PureComponent
- React.useMemo(fn: () => Component, deps) <函数式组件常用>

```javascript

  const ComponentByMemo = React.memo(compoennt, (prevProps, nextProps) => {
    //  如果 props 相等，默认回收函数 会返回 true，并且不更新；如果 props 不相等，则返回 false，组件更新
    // 这与 shouldComponentUpdate 方法的返回值相反， shouldComponent 返回true的话，更新，反正不更新
  } )

  class Levi extends React.PureComponent {
    render() {
      return <div>levis</div>
    }
  }

  const ComponentByUseMemo = React.useMemo()

```

#### 如何提高react性能
- 适当的使用shouldComponentUpdate/React.memo，避免了子组件不必要的渲染
- 在列表和表格中使用key，这样React渲染更新速度更快
- 抽离公共代码到单独文件中，使用路由懒加载
**补充前端性能优化**
- 减少http请求，缓存相关的ajax数据
- 图片优化，压缩图片
- css优化，提取共有css代码
- js优化，提取公共js代码-
- 使用CDN内容分发网络，
- 使用响应的打包工具：webpack,gulp压缩代码
- DOM渲染优化。重绘不一定引起回流，但是回流会重绘，所以在弄DOM渲染的时候，毕竟发生过多的回流以及重绘

#### 什么是React.Fiber

#### react 事件系统

#### react为什么要引入hooks
- 组件之间逻辑复杂难用
- 大型的组件难以拆分
- 难以理解的class语法，忘记事件绑定this，没有稳定的语法提案
相对类组件，函数式组件又过于简陋，没有生命周期钩子，没有类组件的状态。但是函数式组件上手简单，友好直观
```javascript
  import React, {useState} from 'react';

  const Levi = () => {

    const [count, setCount] = useState<number>(0);

    const handleClick = useCallback(() => {
      setCount(count+1)
    }, [])

    return <Button onclick={handleClick}>点击</Button>
  }

```

#### 常用hooks
- useState()状态钩子。为函数组建提供内部状态
  ```javascript
    const Page = () => {
      const [name, setName] = useState<string>('levi')
    };
  ```
- useContext()共享钩子。该钩子的作用是，在组件之间共享状态。 可以解决react逐层通过Porps传递数据，它接受React.createContext()的返回结果作为参数，使用useContext将不再需要Provider 和 Consumer
  ```javascript
    const Page = () => {
      const info = React.useContext(LeviInfoContext);

      return <div>11111{info.name}</div>;
    };
  ```
- useReducer()状态钩子。Action 钩子。useReducer() 提供了状态管理，其基本原理是通过用户在页面中发起action, 从而通过reducer方法来改变state, 从而实现页面和状态的通信。使用很像redux
  ```javascript 
    const initialState = { count: 0 };

    function reducer(state, action) {
      switch (action.type) {
        case 'increment':
          return { count: state.count + 1 };
        case 'decrement':
          return { count: state.count - 1 };
        default:
          throw new Error();
      }
    }

    function Counter() {
      const [state, dispatch] = useReducer(reducer, initialState);
      return (
        <>
          Count: {state.count}
          <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
          <button onClick={() => dispatch({ type: 'increment' })}>+</button>
        </>
      );
    }
  ```
- useEffect()副作用钩子。它接收两个参数， 第一个是进行的异步操作， 第二个是数组，用来给出Effect的依赖项
  ```javascript
    const Page = () => {
      useEffect(() => {
        const initial = async () => {
          // fetch data
          await Promise.resolve();
        }
      }, [])

    return <div>11111{info.name}</div>;
  };
  ```
- useRef()获取组件的实例；渲染周期之间共享数据的存储(state不能存储跨渲染周期的数据，因为state的保存会触发组件重渲染）,useRef传入一个参数initValue，并创建一个对象{ current: initValue }给函数组件使用，在整个生命周期中该对象保持不变
  ```javascript
    const Page = () => {
      const leviRef = React.useRef(initialVal)

      const handleClick = useCallback(() => {
        levi.current.focus();
      }, [])

      return <input ref={leviRef} onClick={handleClick} />
    },
  ```
- useMemo和useCallback可缓存函数的引用或值，useMemo缓存数据(所有对象、包括函数)，useCallback缓存函数，两者是Hooks的常见优化策略，useCallback(fn,deps)相当于useMemo(()=>fn,deps)
  ```javascript
      const Page = () => {
        const leviRef = React.useRef(initialVal)
        
        const handleClick = useCallback(() => {
          levi.current.focus();
        }, [])

        return React.useMemo(() = <input ref={leviRef} onClick={handleClick} />, [])
    },
  ```

#### Hooks相比HOC和Render Prop有哪些优点？
 HOC都属于一种开发模式,将复用逻辑提升至父组件，容易嵌套过多、过度包装
 hooks是react一种api, 将复用逻辑提升组件顶层，而不是提升至父组件中，避免造成多层嵌套

#### hooks 里面有哪些优化方案
- deps依赖项不过过多，尽量不要超出3个
- 多个相关state，合并成对象形式处理，不然会导致hooks内部出现多个state，难以维
- 在 useCallback 内部使用了 setState ，可以考虑使用 setState callback 减少依赖
  ```javascript
    const useValues = () => {
      const [values, setValues] = useState({});

      const updateData = useCallback((nextData) => {
        setValues((prevValues) => ({
          data: nextData,
          count: prevValues.count + 1,    
        })); // 通过 setState 回调函数获取最新的 values 状态，这时 callback 不再依赖于外部的 values 变量了，因此依赖数组中不需要指定任何值
      }, []); // 这个 callback 永远不会重新创建

      return [values, updateData];
    };

- 可有ref来保存变量(组件整个生命周期ref会保持不变)
- 编写自定义hooks返回可以选择元祖类型，如果超出3个，建议考虑对象形式返回
- 自定义hooks返回的值，要保持引用的一致性
  ```javascript
    function Example() {
      const data = useData();
      const [dataChanged, setDataChanged] = useState(false);

      useEffect(() => {
        setDataChanged((prevDataChanged) => !prevDataChanged); 
        // 当 data 发生变化时，调用 setState。
        // 如果 data 值相同而引用不同，就可能会产生非预期的结果。 
      }, [data]);

      console.log(dataChanged);

      return <LeviComponent data={data} />;
    }

    const useData = () => {
      // 获取异步数据  
      const resp = getAsyncData([]);

      // 处理获取到的异步数据，这里使用了 Array.map。
      // 因此，即使 data 相同，每次调用得到的引用也是不同的。  
      const mapper = (data) => data.map((item) => ({...item, selected: false}));

      return resp ? mapper(resp) : resp;
    };
  ```

#### useCallback 是用来干嘛的
缓存函数

#### useEffect和useLayoutEffect区别？
1、useEffect是render结束后，callback函数执行，但是不会阻断浏览器的渲染。
2、useLayoutEffect里面的callback函数会在DOM更新完成后立即执行,但是会在浏览器进行任何绘制之前运行完成,阻塞了浏览器的绘制。

#### react新版生命周期getDrivedStatefromprops为什么是静态的
react 16.4版本引入，代替换来的componentWillReceiveProps, 当props或state改变的时候，触发，是一个纯函数。
之所以设置为静态，在函数内部不能获取this，防止setState造成死循环; 

#### 实现一个自定义hooks
注意： 自定义hooks以use开头、或者函数名开头大写(react会认为这是一个组件)。不会react-hooks校验规则会报错

```javascript
  // 获取当前文档标题 && 设置文档标题
  import { useState, useEffect, useCallback } from 'react';
  const useTitle = () => {
    const [title, setTitle] = useState<string>(document.title);

    useEffect(() => {
      document.title = title;
    }, [title]);

    const setDocumentTitle = useCallback(newTitle => {
      setTitle(newTitle);
    }, []);

    return [title, setDocumentTitle];
  };

  export default useTitle;

  // fetch data
  import { useState, useCallback } from 'react';
  const useFetchCity = () => {
    const [city, setCity] = useState<any[]>([]);
    const [loading, setLoading] = useState<boolean>(false);

    const getAllCity = useCallback(async () => {
      setLoading(true);

      const res = await new Promise(resolve => {
        setTimeout(() => {
          resolve([1, 2, 3, 4]);
        }, 2000);
      });

      setCity(res);
      setLoading(false);
    }, []);

    return [loading, city, getAllCity];
  };

  export default useFetchCity;


```