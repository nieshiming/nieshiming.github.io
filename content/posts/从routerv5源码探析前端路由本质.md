+++ 
draft = true
date = 2021-12-15
title = "routerv5源码探析前端路由本质"
description = ""
slug = ""
authors = []
tags = ["js", "react"]
categories = ["react"]
externalLink = ""
series = []
+++

## 前言

### 什么是前端路由
首先，路由的概念开始是后端提出来的，用来跟服务器进行数据/资源获取的一种方式，通过不同的路径，来获取不同的资源
前端随着ajax(不刷新页面情况下请求数据)的流行，推动着异步交互体验提升，随后spa<单页面应用程序>在前端领域大放异彩
> spa: 单页面应用程序不仅在页面内交互是无刷新的，连页面之间的跳转也没有刷新。

spa核心思想
- 监听url变化
- 改变context的值
- 匹配相对应的组件

### 实现前端路由应该包括哪些功能
前端可以自己维护和控制浏览器的history<历史记录>。我们称之为history栈， 保证浏览器在url改变的时候不会刷新页面。并且通过history栈控制浏览器页面的前进和后退

目前 Router有两种实现方式  hash 和 History

    ...
<!--more-->

#### Hash
hash表示页面中的一个位置，当浏览器页面完全加载好了，页面会滚动到hash位置指定的地。
- hash 只作用在浏览器，不会在请求中发送给服务器
- hash 发生变化时，浏览器并不会重新给后端发送请求加载页面。
- hash 发生变化时会触发 hashchange 事件，在该事件中可以通过 window.location.hash 获取到当前 hash值

#### History
在 html5 中新增了 history.pushState() 和 history.replaceState()，相比hash路由 url上不美观的hash值 ，取而代之使用 history.pushState 来完成对 window.location 的操作。

#### History和Hash对比
- hash后面使用#来模拟完整的路径，不太美观
- 用户手动刷新页面, 对于hashRouter来说，后端收到的是同一个地址， history因为直接修改浏览器url，对于后端而言接受不同的地址，需要后端对资源做统一跳转处理<nginx>
webpack本地开发模式下，用webpack-dev-server插件开启本地服务器，解决请求资源: **historyapifallback**

### React-Router
React作为前端视图层的框架，本身是不具有除了view层面以外的功能。需要引入React-Router, 对 react的来说管理路由需要管理组件的生命周期，对不同的路由渲染不同的组件。

React-Router 库包含三个包：react-router、react-router-dom 和 react-router-native 。**路由操作的核心包 react-router**，而其他两个是特定环境下使用的。如果我们开发web应用，使用react-router-dom，如果开发RN相关react-router-native。

React-Router实现单页面应用程序路由跳转，分为HashRouter, BrowserRouter
**browserHistory 是使用 React-Router 的应用推荐的 history方案**

## 学前小知识
-  [html5 history.pushstate && window.popstate](https://developer.mozilla.org/zh-CN/docs/Web/API/WindowEventHandlers/onpopstate)
- [React.Context](https://zh-hans.reactjs.org/docs/context.html)
- 发布订阅设计模式

### history.pushstate
> history.pushState方法向当前浏览器会话的历史堆栈中添加一个状态 **history.pushState(state, title[, url])**
- state: state状态是一个JavaScript对象, 每当用户导航到新状态时，都会触发popstate事件，并且该事件的状态属性指向了创建历史条目的创建的state属性
- title: 大多数浏览器都会忽略这个属性
- url: 新历史记录条目的URL由此参数指定, "新的url必须和当前网址同源" 

### window.popstate
> 当活动的历史记录更改的时候，将会触发popstate事件，如果被激活的历史记录条目是通过对history.pushState()的调用创建的，或者受到对history.replaceState()的调用。popstate事件的state属性包含历史条目的状态对象的副本。

histry.pushstate、history.replacestate方法调用不会触发window.popstate事件，只要作为浏览器行为才会触发、例如操作哦了tab页上面前进和后退，或者调用了history.back()、history.forword();

## 源码分析
- react-router-dom
- react-router
- history

### react-router-dom
react-router-dom在react-router核心基础上扩展了可操作DOM的api， 添加了用于跳转的Link组件、history模式下的BrowserRouter组件和hash模式下的HashRouter组件。
**BrowserRouter和HashRouter，调用了history库中createBrowserHistory和createHashHistory方法**

### react-router
react路由核心包。 提供了路由的核心组件。如Router、Route、Switch等，但没有提供有关dom操作进行路由跳转的api；

### history
可以理解为react-router的核心，也是整个路由原理的核心，里面集成了底层路由原理的实现。

### BrowserRouter源码分析

#### BrowserRouter
内部创建了一个全局的history对象<用于监听整个路由的变化>， 并且把history作为props传递的react-router的Router组件

```js
class BrowserRouter extends React.Component {
  history = createHistory(this.props);

  render() {
    return <Router history={this.history} children={this.props.children} />;
  }
}
```

#### Router
- 构造函数中把history.location作为自己的state，并且监听了location的变化。
- render中利用了React的Context提供了RouterContext，HistoryContext两个Context信息，供子元素使用。

```javascript

class Router extends React.Component {
  static computeRootMatch(pathname) {
    return { path: "/", url: "/", params: {}, isExact: pathname === "/" };
  }

  constructor(props) {
    super(props);

    this.state = {  
      location: props.history.location
    };

    // This is a bit of a hack. We have to start listening for location
    // changes here in the constructor in case there are any <Redirect>s
    // on the initial render. If there are, they will replace/push when
    // they mount and since cDM fires in children before parents, we may
    // get a new location before the <Router> is mounted.

    // 这有点hack。 我们必须开始在构造函数中监听位置更改，以防初始渲染中存在任何<Redirect>。
    // 如果有的话，它们将在安装时替换/推动，并且由于cDM在父级之前在子级中触发，
    // 因此在<Router>挂在之前，我们可能会获得一个新位置

    // 因为子组件会比父组件更早渲染完成, 以及<Redirect>的存在, 
    // 若是在<Router>的componentDidMount生命周期中对history.location进行监听, 则有可能在监听事件注册之前,
    // history.location已经由于<Redirect>发生了多次改变, 因此我们需要在<Router>的constructor中就注册监听事件

    // 判断组件是否加载完成
    this._isMounted = false;
    this._pendingLocation = null;

    // 如果不是服务端渲染，监听history变更
    if (!props.staticContext) {
      // 每次路由变化 -> 触发顶层 Router 的回调事件 -> Router 进行 setState -> 向下传递 nextContext（context 中含有最新的 location）
      this.unlisten = props.history.listen(location => {
        // 组件未加载完毕，但是 location 发生的变化，暂存在 _pendingLocation 字段中
        if (this._isMounted) {
          this.setState({ location });
        } else {
          this._pendingLocation = location;
        }
      });
    }
  }

  componentDidMount() {
    this._isMounted = true;

    if (this._pendingLocation) {
      this.setState({ location: this._pendingLocation });
    }
  }

  //  卸载监听器
  componentWillUnmount() {
    if (this.unlisten) this.unlisten();
  }

  /**
   * @description
   * Router中的2个context由HistoryContext和RouterContext组件。 考虑到兼容性，并没有使用 React.createContext 方式  来创建
   * 
   * */ 

  render() {
    return (
      <RouterContext.Provider
        value={{
          history: this.props.history,
          location: this.state.location,
          // 解析得到 包含path url params isExact 四个属性的属性。 默认指向了根地址
          match: Router.computeRootMatch(this.state.location.pathname),
          // 只有StaticRouter会传staticContext用于服务端渲染。 HashRouter 和 BrowserRouter 都是 null
          staticContext: this.props.staticContext
        }}
      >
        <HistoryContext.Provider
          children={this.props.children || null}
          value={this.props.history}
        />
      </RouterContext.Provider>
    );
  }
}


```


#### Route
Route 组件根据自身的传参，对上层 RouterContext 中的部分属性（location 和 match）进行了更新，并且如果当前路径和配置的 path 路径 match，则渲染该组件，渲染的方式有 children，component，render 三种方式，我们最常用的就是 component 方式，注意每种方式的区别

- Route的component，render，children三个属性是互斥的
- 优先级children>component>render
- children在无论路由匹配与否，都会渲染

```javascript

class Route extends React.Component {
  render() {
    return (
      // 获取从RouterContext共享的context上下文
      <RouterContext.Consumer>
        {context => {
          invariant(context, "You should not use <Route> outside a <Router>");

          const location = this.props.location || context.location;
          const match = this.props.computedMatch
            ? this.props.computedMatch // <Switch> already computed the match for us
            : this.props.path
            ? matchPath(location.pathname, this.props)
            : context.match;

          const props = { ...context, location, match };

          // 提供3种渲染组件的方式
          let { children, component, render } = this.props;

          if (Array.isArray(children) && children.length === 0) {
            children = null;
          }

         // 渲染逻辑
         // 当props匹配了路由时，先判断是否匹配，如果不匹配就将props向下传递。
          return (
            <RouterContext.Provider value={props}>
              {props.match
                ? children
                  ? typeof children === "function"
                    ? __DEV__
                      ? evalChildrenDev(children, props, this.props.path)
                      : children(props)
                    : children
                  : component
                  ? React.createElement(component, props)
                  : render
                  ? render(props)
                  : null
                : typeof children === "function"
                ? __DEV__
                  ? evalChildrenDev(children, props, this.props.path)
                  : children(props)
                : null}
            </RouterContext.Provider>
          );
        }}
      </RouterContext.Consumer>
    );
  }
}

```

组件渲染逻辑如下：

![image.png](https://pic3.zhimg.com/80/v2-50844a678c2d16134bf5d54f6addeb20_1440w.png)

#### Switch

switch 用来嵌套在Route外面，当Switch中的第一个Route匹配后就不会渲染其他的Route了
**案例见switch.tsx**

```javascript
class Switch extends React.Component {
  render() {
    return (
      <RouterContext.Consumer>
        {context => {
          invariant(context, "You should not use <Switch> outside a <Router>");

          const location = this.props.location || context.location;

          let element, match;


          // React.Children.forEach 对子元素做遍历
          React.Children.forEach(this.props.children, child => {
            if (match == null && React.isValidElement(child)) {
              element = child;

              // from具体是<Redirect /> 使用
              const path = child.props.path || child.props.from;

              // 判断组件是否匹配
              match = path
                ? matchPath(location.pathname, { ...child.props, path })
                : context.match;
            }
          });

          return match
            ? React.cloneElement(element, { location, computedMatch: match })
            : null;
        }}
      </RouterContext.Consumer>
    );
  }
}

export default Switch;

```

```javascript

const cache = {};
const cacheLimit = 10000;
let cacheCount = 0;

function compilePath(path, options) {
  // 做一个全局缓存，确保计算出来的结果能够得到复用
  const cacheKey = `${options.end}${options.strict}${options.sensitive}`;
  const pathCache = cache[cacheKey] || (cache[cacheKey] = {});

  if (pathCache[path]) return pathCache[path];

  const keys = [];
  // 将字符串路径转化成为表达式
  const regexp = pathToRegexp(path, keys, options);
  const result = { regexp, keys };

  // 做多缓存 10000 个
  if (cacheCount < cacheLimit) {
    pathCache[path] = result;
    cacheCount++;
  }

  return result;
}


function matchPath(pathname, options = {}) {
  // 规范结构体
  if (typeof options === "string" || Array.isArray(options)) {
    options = { path: options };
  }

  const { path, exact = false, strict = false, sensitive = false } = options;

  // 转化成数组进行判断
  const paths = [].concat(path);

  return paths.reduce((matched, path) => {
    if (!path && path !== "") return null;
    if (matched) return matched;

    // exact: 如果为 true，则只有在路径完全匹配 location.pathname 时才匹配。 
    // strict: 在确定为位置是否与当前 URL 匹配时，将考虑位置 pathname 后的斜线。
    // sensitive: 如果路径区分大小写，则为 true ，则匹配
    const { regexp, keys } = compilePath(path, {
      end: exact,
      strict,
      sensitive
    });
    const match = regexp.exec(pathname);

    if (!match) return null;

    const [url, ...values] = match;
    const isExact = pathname === url;

    if (exact && !isExact) return null;

    return {
      path, // the path used to match
      url: path === "/" && url === "" ? "/" : url, // the matched portion of the URL
      isExact, // whether or not we matched exactly
      params: keys.reduce((memo, key, index) => {
        memo[key.name] = values[index];
        return memo;
      }, {})
    };
  }, null);
}

```

#### Link

通过审查元素发现，link最终后悔创建一个a标签来包裹要跳转的元素的元素，但是如果只是一个普通的带 href 的 a 标签，那么就会直接跳转到一个新的页面而不是 SPA 了。所以a标签中的默认跳转事件会被禁止调。所以这里的 href 并没有实际的作用，但仍然可以标示出要跳转到的页面的 URL 并且有更好的 html 语义

对没有被 “preventDefault调用 && 鼠标左键点击的 && 非 _blank 跳转 的&& 没有按住其他功能键的“ 单击进行 preventDefault，然后 push 进 history 中，这也是前面讲过的 —— 路由的变化 与 页面的跳转 是不互相关联的，react-router中  Link 中通过 history 库的 push 调用了 H5 history 的 pushState，但是这仅仅会让路由变化，其他什么都没有改变。  之前在Router创建 listen，它会监听路由的变化，然后通过 context 更新 props 和 nextContext 让下层的 Route 去重新匹配，完成需要渲染部分的更新


点击时候进行如下判断。当下面4个条件都满足调用navigate方法。否则新窗口打开
- event.defaultPrevented: 返回一个boolean，表明当前事件是否调用了event.preventDefault()
- event.button === 0 鼠标左键
- target === "_self" 非_blank跳转
- !isModifiedEvent: 点击事件发生时候，没有同时按住metaKey， altKey， ctrlKey， shiftKey

[Refs转发](https://zh-hans.reactjs.org/docs/forwarding-refs.html#forwarding-refs-in-higher-order-components)

```javascript

function isModifiedEvent(event) {
  return !!(event.metaKey || event.altKey || event.ctrlKey || event.shiftKey);
}

/ React 15 compat
const forwardRefShim = C => C;
let { forwardRef } = React;
if (typeof forwardRef === "undefined") {
  forwardRef = forwardRefShim;
}

function isModifiedEvent(event) {
  return !!(event.metaKey || event.altKey || event.ctrlKey || event.shiftKey);
}

const LinkAnchor = forwardRef(
  (
    {
      innerRef, // TODO: deprecate
      navigate,
      onClick,
      ...rest
    },
    forwardedRef
  ) => {
    const { target } = rest;

    let props = {
      ...rest,
      onClick: event => {
        try {
          if (onClick) onClick(event);
        } catch (ex) {
          event.preventDefault();
          throw ex;
        }

        if (
          !event.defaultPrevented && // onClick prevented default
          event.button === 0 && // ignore everything but left clicks
          (!target || target === "_self") && // let browser handle "target=_blank" etc.
          !isModifiedEvent(event) // ignore clicks with modifier keys
        ) {
          
          // event.preventDefault()阻止超链接默认事件, 避免点击<Link>后重新刷新页面;
          event.preventDefault();
          navigate();
        }
      }
    };

    if (forwardRefShim !== forwardRef) {
      props.ref = forwardedRef || innerRef;
    } else {
      props.ref = innerRef;
    }

    // 渲染了一个没有默认跳转行为a标签，跳转行为由navigate实现
    return <a {...props} />;
  }
);


/**
 * The public API for rendering a history-aware <a>.
 */
const Link = forwardRef(
  (
    {
      component = LinkAnchor,
      replace,
      to,
      innerRef, // TODO: deprecate
      ...rest
    },
    forwardedRef
  ) => {
    return (
      <RouterContext.Consumer>
        {context => {
          invariant(context, "You should not use <Link> outside a <Router>");

          const { history } = context;

         // 生成location对象
          const location = normalizeToLocation(
            resolveToLocation(to, context.location),
            context.location
          );

          // 拼接完整路径 basename+path
          const href = location ? history.createHref(location) : "";
          const props = {
            ...rest,
            href,
            navigate() {
              const location = resolveToLocation(to, context.location);
              const method = replace ? history.replace : history.push;

              // 执行history.push 或者 history.replace 默认 push
              method(location);
            }
          };

          if (forwardRefShim !== forwardRef) {
            props.ref = forwardedRef || innerRef;
          } else {
            props.innerRef = innerRef;
          }

          return React.createElement(component, props);
        }}
      </RouterContext.Consumer>
    );
  }
);

```


#### withRouter
withRouter是一个[高阶组件](https://zh-hans.reactjs.org/docs/higher-order-components.html)， 可以让普通非包裹在Route的组件也能获取路由信息。把react-router 的 history、location、match 三个对象传入 props上

> 高阶组件： 高阶组件(HOC)是React中用于复用组件逻辑的一种高级技巧、自己不是React Api的一部分， 是基础React组合特性行为的设计模式


```javascript

function withRouter(Component) {
  const displayName = `withRouter(${Component.displayName || Component.name})`;
  const C = props => {
    const { wrappedComponentRef, ...remainingProps } = props;

    return (
      <RouterContext.Consumer>
        {context => {
          invariant(
            context,
            `You should not use <${displayName} /> outside a <Router>`
          );

          {/* 把context注入到Component中 */}
          return (
            <Component
              {...remainingProps}
              {...context}
              ref={wrappedComponentRef}
            />
          );
        }}
      </RouterContext.Consumer>
    );
  };

  C.displayName = displayName;
  C.WrappedComponent = Component;

  if (__DEV__) {
    C.propTypes = {
      wrappedComponentRef: PropTypes.oneOfType([
        PropTypes.string,
        PropTypes.func,
        PropTypes.object
      ])
    };
  }

  /**
   * @description 
   * 当给组件添加至高阶组件中后，原来的组件会被一组容器组件包含。这样意味着。容器组件不会有原来组件的任何的静态方法
   * 为了解决这个问题， 在返回容器组件之前。务必复制wrappedComponent的静态方法到容器组件上
   * */ 
  return hoistStatics(C, Component);
}


```
有时候React组件定义的静态很有有用，[务必复制静态方法](https://zh-hans.reactjs.org/docs/higher-order-components.html#static-methods-must-be-copied-over)


#### Redirect
该组件在componentDidMount生命周期内，通过history Api跳转到path指定位置， 默认情况下，新位置将覆盖历史堆栈中的当前位置。

```javascript
function Redirect({ computedMatch, to, push = false }) {
  return (
    <RouterContext.Consumer>
      {context => {
        invariant(context, "You should not use <Redirect> outside a <Router>");

        const { history, staticContext } = context;

        const method = push ? history.push : history.replace;
        // to 重定向地址，可以是一个string， 也可以是对象

        // computedMatch 从switch上面拿到match
        const location = createLocation(
          computedMatch
            ? typeof to === "string"
              ? generatePath(to, computedMatch.params)
              : {
                  ...to,
                  pathname: generatePath(to.pathname, computedMatch.params)
                }
            : to
        );

        // 服务端渲染直接执行方法一次
        if (staticContext) {
          method(location);
          return null;
        }

        // lifeCycle不会渲染任何页面， 只有一些生命周期函数componentDidMount、componentDidUpdate、componentWillUnmount
        return (
          <Lifecycle
            onMount={() => {
              method(location);
            }}
            // componentDidUpdate 时候判断当前 location 和上一个 location 是否发生变化
            // 一般来说在componentDidMount就调走了。不会走到ComponentDidUpdate
            onUpdate={(self, prevProps) => {
              const prevLocation = createLocation(prevProps.to);
              if (
                !locationsAreEqual(prevLocation, {
                  ...location,
                  key: prevLocation.key
                })
              ) {
                method(location);
              }
            }}
            to={to}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}
```

#### Prompt
用于路由切换提示，在某些场景下比较有用，比较用户咋某个页面修改数据。离开页面的时候，提示用户是保存。 详细在history中block明确解释
- message：用于显示提示的文本信息。
- when：默认是 true，设置成 false 时，失效

Prompt 的本质是在 when 为 true 的时候，调用 context.history.block 方法，为全局注册路由监听。**见prompt.tsx**

```javascript
function Prompt({ message, when = true }) {
  return (
    <RouterContext.Consumer>
      {context => {
        invariant(context, "You should not use <Prompt> outside a <Router>");

        if (!when || context.staticContext) return null;

        // 调用history.block注册全局路由监听器
        // message可以是字符串也可以是一个函数， 如果是字符串默认调用window.confirm方法
        // 如果是一个函数，需要返回一个boolean判断是否需要拦截
        const method = context.history.block;

        return (
          <Lifecycle
            onMount={self => {
              // 调用了 history.block 方法
              self.release = method(message);
            }}
            onUpdate={(self, prevProps) => {
              if (prevProps.message !== message) {
                self.release();
                self.release = method(message);
              }
            }}
            onUnmount={self => {
              self.release();
            }}
            message={message}
          />
        );
      }}
    </RouterContext.Consumer>
  );
}

```

#### hooks
hooks是react16.8引入的特性，允许我们在不写class的情况下，操作state和react其他特新。为了在hooks即函数式组件能够操作路由。react-router提供了hooks方法， 底层都是使用React.useContext api <可以获取指定context的值>
- useHistory: 返回一个history对象
- useLocation 返回context下的location对象
- useParams 返回当前匹配路径的params
- useRouteMatch   可以有一个参数 path，如果什么都不传，会返回当前 context 上的 match 的值。 如果传了 path，会比较这个 path 和当前 location 是否 match

```javascript
const useContext = React.useContext;

export function useHistory() {
  if (__DEV__) {
    invariant(
      typeof useContext === "function",
      "You must use React >= 16.8 in order to use useHistory()"
    );
  }

  return useContext(HistoryContext);
}

export function useLocation() {
  if (__DEV__) {
    invariant(
      typeof useContext === "function",
      "You must use React >= 16.8 in order to use useLocation()"
    );
  }

  return useContext(Context).location;
}

export function useParams() {
  if (__DEV__) {
    invariant(
      typeof useContext === "function",
      "You must use React >= 16.8 in order to use useParams()"
    );
  }

  const match = useContext(Context).match;
  return match ? match.params : {};
}

export function useRouteMatch(path) {
  if (__DEV__) {
    invariant(
      typeof useContext === "function",
      "You must use React >= 16.8 in order to use useRouteMatch()"
    );
  }

  const location = useLocation();
  const match = useContext(Context).match;

  return path ? matchPath(location.pathname, path) : match;
}

```

#### 小节
- Router初始化，创建监听函数， 底层逻辑全部由history库管理
- 当点击Link标签的时候，实际上点击是页面渲染出来的a标签。通过preventDefault来组织a标签的页面跳转
- Link中拿到context传递的history， 进行路由跳转
- 路由发生变化，触发了监听函数，Router会重新setState, 每次路由变化 -> 触发顶层 Router 的监听事件 -> Router 触发 setState -> 向下传递新的 nextContext
- 下层Route组件拿到最新nextContext后判断当前path和location是否匹配。内置component，render，children三个属性的渲染机制，并且通过switch组件匹配唯一的路由

### history库源码分析
原理就是封装了原生的html5 的history api, 如pushState, replaceState。当这些事件被触发的时候，执行响应回调函数，用于操控和观察地址栏的变更

history库创建了一个虚拟的history对象， 操纵浏览器地址变更，或者操作hash变更、管理内存中的虚拟历史堆栈

#### utils
pathUtils
```javascript
  // 对传递的path首部添加/
  function addLeadingSlash(path) {
    return path.charAt(0) === '/' ? path : '/' + path;
  }
  // 对传递path去掉首部的/
  function stripLeadingSlash(path) {
    return path.charAt(0) === '/' ? path.substr(1) : path;
  }
  // 判断path中是否包含basename
  function hasBasename(path, prefix) {
    return path.toLowerCase().indexOf(prefix.toLowerCase()) === 0 && '/?#'.indexOf(path.charAt(prefix.length)) !== -1;
  }
  // 如果传递了pathname， 把path中首部basename去掉
  function stripBasename(path, prefix) {
    return hasBasename(path, prefix) ? path.substr(prefix.length) : path;
  }
  // 去掉尾部的/
  function stripTrailingSlash(path) {
    return path.charAt(path.length - 1) === '/' ? path.slice(0, -1) : path;
  }



  // 在createLocation调用
  // 把字符串路径path解析成 {pathname, search, hash}的对象返回出去
  function parsePath(path) {
    var pathname = path || '/';
    var search = '';
    var hash = '';
    var hashIndex = pathname.indexOf('#');

    if (hashIndex !== -1) {
      hash = pathname.substr(hashIndex);
      pathname = pathname.substr(0, hashIndex);
    }

    var searchIndex = pathname.indexOf('?');

    if (searchIndex !== -1) {
      search = pathname.substr(searchIndex);
      pathname = pathname.substr(0, searchIndex);
    }

    return {
      pathname: pathname,
      search: search === '?' ? '' : search,
      hash: hash === '#' ? '' : hash
    };
  }
  

  // 把location对象<{pathname, search, hash}> 生成最终的地址栏路径
  function createPath(location) {
    var pathname = location.pathname,
        search = location.search,
        hash = location.hash;
    var path = pathname || '/';
    if (search && search !== '?') path += search.charAt(0) === '?' ? search : "?" + search;
    if (hash && hash !== '#') path += hash.charAt(0) === '#' ? hash : "#" + hash;
    return path;
  }

```

#### domUtils
```javascript

  // 是否可以操作DOM节点，即判断window.document对象是否存在
  var canUseDOM = !!(typeof window !== 'undefined' && window.document && window.document.createElement);

  // 路由跳转拦截回调函数，默认使用window.confirm
  function getConfirmation(message, callback) {
    callback(window.confirm(message)); // eslint-disable-line no-alert
  }
  /**
   * Returns true if the HTML5 history API is supported. Taken from Modernizr.
   *
   * https://github.com/Modernizr/Modernizr/blob/master/LICENSE
   * https://github.com/Modernizr/Modernizr/blob/master/feature-detects/history.js
   * changed to avoid false negatives for Windows Phones: https://github.com/reactjs/react-router/issues/586
   */

  // 不支持 安卓是2. 和 4.0版本 并且ua信息包含 ’Mobile Safari‘ && Chrome &&  Windows Phone 
  // 判断主流浏览器平台是否支持html5 history api
  function supportsHistory() {
    var ua = window.navigator.userAgent;
    if ((ua.indexOf('Android 2.') !== -1 || ua.indexOf('Android 4.0') !== -1) && ua.indexOf('Mobile Safari') !== -1 && ua.indexOf('Chrome') === -1 && ua.indexOf('Windows Phone') === -1) return false;
    return window.history && 'pushState' in window.history;
  }
  /**
   * Returns true if browser fires popstate on hash change.
   * IE10 and IE11 do not.
   */

  // 判断主流浏览器平台在hashchange的时候是否会触发popstate 事件， IE10,IE10并不会
  function supportsPopStateOnHashChange() {
    return window.navigator.userAgent.indexOf('Trident') === -1;
  }
  /**
   * Returns false if using go(n) with hash history causes a full page reload.
   */

  // 当使用go变更hash的时候，会不会造成页面刷新
  function supportsGoWithoutReloadUsingHash() {
    return window.navigator.userAgent.indexOf('Firefox') === -1;
  }
  /**
   * Returns true if a given popstate event is an extraneous WebKit event.
   * Accounts for the fact that Chrome on iOS fires real popstate events
   * containing undefined state when pressing the back button.
   */
  // 判断popstate是否是有效的
  // 如果给定的popstate事件是无关的webkit事件， 则会返回true,
  // 在IOS上chrome会触发state为undefined 真实的popstate事件
  function isExtraneousPopstateEvent(event) {
    return event.state === undefined && navigator.userAgent.indexOf('CriOS') === -1;
  }

  var PopStateEvent = 'popstate';
  var HashChangeEvent = 'hashchange';

  // 返回history的state
  function getHistoryState() {
    try {
      // state 必须有pustate/replaceState产生，不然都是null
      return window.history.state || {};
    } catch (e) {
      // IE11 下有时候会抛出异常， 返回返回的state的是一个对象
      return {};
    }
  }
  ```

createLocation

生成 {pathname, search, hash, state, key} 对象

```javascript
  function createLocation(path, state, key, currentLocation) {
    var location;

    if (typeof path === 'string') {
      // 分解{pathname, search, hash}对象
      location = parsePath(path);
      // 添加state属性到location上
      location.state = state;
    } else {
      location = _extends({}, path);

      // 补足location操作
      if (location.pathname === undefined) location.pathname = '';

      if (location.search) {
        if (location.search.charAt(0) !== '?') location.search = '?' + location.search;
      } else {
        location.search = '';
      }

      if (location.hash) {
        if (location.hash.charAt(0) !== '#') location.hash = '#' + location.hash;
      } else {
        location.hash = '';
      }

      if (state !== undefined && location.state === undefined) location.state = state;
    }

    try {
      // 尝试对pathname decodeURI 解码
      location.pathname = decodeURI(location.pathname);
    } catch (e) {
      if (e instanceof URIError) {
        // decodeURI() 函数能解码由encodeURI 创建或其它流程得到的统一资源标识符（URI）。
        // decodeURI在解析非合法的URI编码是抛出URIError类型错误
        throw new URIError('Pathname "' + location.pathname + '" could not be decoded. ' + 'This is likely caused by an invalid percent-encoding.');
      } else {
        throw e;
      }
    }

    if (key) location.key = key;

    if (currentLocation) {
      // Resolve incomplete/relative pathname relative to current location.
      if (!location.pathname) {
        location.pathname = currentLocation.pathname;
      } else if (location.pathname.charAt(0) !== '/') {
        location.pathname = resolvePathname(location.pathname, currentLocation.pathname);
      }
    } else {
      // When there is no prior location and pathname is empty, set it to /

      // 如果pathname为空并且，没有currentLocation 定向到根节点
      if (!location.pathname) {
        location.pathname = '/';
      }
    }

    return location;
  }

```

#### createTransitionManager
创建任务管理器=> 控制路由跳转以及添加路由监听函数
```javascript
  function createTransitionManager() {
    var prompt = null;

    // 这里使用了闭包，设置了prompt,返回了清空函数， prompt可以是一个string或者函数
    function setPrompt(nextPrompt) {
      process.env.NODE_ENV !== "production" ? warning(prompt == null, 'A history supports only one prompt at a time') : void 0;
      prompt = nextPrompt;
      return function () {
        if (prompt === nextPrompt) prompt = null;
      };
    }

    function confirmTransitionTo(location, action, getUserConfirmation, callback) {
      // TODO: If another transition starts while we're still confirming
      // the previous one, we may end up in a weird state. Figure out the
      // best way to handle this.

      // 是否设置路由跳转拦截器
      if (prompt != null) {
        var result = typeof prompt === 'function' ? prompt(location, action) : prompt;

        if (typeof result === 'string') {
          if (typeof getUserConfirmation === 'function') {
            getUserConfirmation(result, callback);
          } else {
            process.env.NODE_ENV !== "production" ? warning(false, 'A history needs a getUserConfirmation function in order to use a prompt message') : void 0;
            callback(true);
          }
        } else {
          // Return false from a transition hook to cancel the transition.
          callback(result !== false);
        }
      } else {
        // 不存在pprompt，直接执行回调函数， 通知notifyListeners
        callback(true);
      }
    }

    // 发布订阅模式，将回调函数加入到listners
    // 存储监听函数
    var listeners = [];

    // 返回取消监听函数
    function appendListener(fn) {
      var isActive = true;

      function listener() {
        if (isActive) fn.apply(void 0, arguments);
      }

      listeners.push(listener);
      return function () {
        isActive = false;
        listeners = listeners.filter(function (item) {
          return item !== listener;
        });
      };
    }

    // 通知被订阅事件开始执行
    function notifyListeners() {
      for (var _len = arguments.length, args = new Array(_len), _key = 0; _key < _len; _key++) {
        args[_key] = arguments[_key];
      }

      // 依次遍历执行监听器数组每个注册的事件
      listeners.forEach(function (listener) {
        return listener.apply(void 0, args);
      });
    }

    return {
      setPrompt: setPrompt,
      confirmTransitionTo: confirmTransitionTo,
      appendListener: appendListener,
      notifyListeners: notifyListeners
    };
  }
```

#### createBrowserHistory
```javascript

 if (props === void 0) {
    props = {};
  }

  !canUseDOM ? process.env.NODE_ENV !== "production" ? invariant(false, 'Browser history needs a DOM') : invariant(false) : void 0;
   // 拿到全局的history对象
  var globalHistory = window.history;

  // 不支持 安卓是2. 和 4.0版本 并且ua信息包含 ’Mobile Safari‘ && Chrome &&  Windows Phone 
  var canUseHistory = supportsHistory();

  // 当hash改变时，如果不能触发popstate事件，则添加hashchange事件当hash改变时，如果不能触发popstate事件，则添加hashchange事件
  var needsHashChangeListener = !supportsPopStateOnHashChange();
  var _props = props,
      _props$forceRefresh = _props.forceRefresh,
       // 默认切换路由不刷新
      forceRefresh = _props$forceRefresh === void 0 ? false : _props$forceRefresh,
      // 初始化是否注入getUserConfirmation函数，默认是window.confirm
      _props$getUserConfirm = _props.getUserConfirmation,
      getUserConfirmation = _props$getUserConfirm === void 0 ? getConfirmation : _props$getUserConfirm,
      _props$keyLength = _props.keyLength,
      // 默认6位长度随机key
      keyLength = _props$keyLength === void 0 ? 6 : _props$keyLength;

      // 添加basename 同时首部添加/ 去掉尾部/ eg: /ahs/xxx
  var basename = props.basename ? stripTrailingSlash(addLeadingSlash(props.basename)) : ''; 

  function getDOMLocation(historyState) {
    // 获取history对象的key和state
    var _ref = historyState || {},
        key = _ref.key,
        state = _ref.state;

    var _window$location = window.location,
        pathname = _window$location.pathname,
        search = _window$location.search,
        hash = _window$location.hash;
        
    // 拼一下完整的路径
    var path = pathname + search + hash;
    process.env.NODE_ENV !== "production" ? warning(!basename || hasBasename(path, basename), 'You are attempting to use a basename on a page whose URL path does not begin ' + 'with the basename. Expected path "' + path + '" to begin with "' + basename + '".') : void 0;

    // 去掉path中的basename
    if (basename) path = stripBasename(path, basename);

    // 生成一个自定义location对象
    return createLocation(path, state, key);
  }

  // 创建36进制的随机数key 从第2位开始截取
  function createKey() {
    return Math.random().toString(36).substr(2, keyLength);
  }

  var transitionManager = createTransitionManager();

  // 路由发生改变的时候，更新history的部分属性，如action, location等，在路由完成跳转后
  // 通知transitionManager触发所有监听函数
  function setState(nextState) {
    _extends(history, nextState);

    // 更新history的length, 实实保持和window.history.length 同步
    history.length = globalHistory.length;

    // 推送至订阅者，执行响应回调函数
    transitionManager.notifyListeners(history.location, history.action);
  }

  // 监听popState事件进行处理<过滤掉IOS上无效的popstate事件，即： state为undefined>
  function handlePopState(event) {
    if (isExtraneousPopstateEvent(event)) return;
  
    // 拿到当前地址的event.state 传递给getDOMLocation。得到最新location对象
    handlePop(getDOMLocation(event.state));
  }

  function handleHashChange() {
    // 监听到hashchange时进行的处理，由于hashchange不会更改state
    // 此处不需要更新location的state
    handlePop(getDOMLocation(getHistoryState()));
  }

  // 是否强制路由加载
  var forceNextPop = false;

   // handlePop是对使用go方法来回退或者前进时，对页面进行的更新，正常情况下来说没有问题
   // 但是如果页面使用Prompt，即路由拦截器。当点击回退或者前进就会触发histrory的api,改变了地址栏的路径
   // 然后弹出需要用户进行确认的提示框，如果用户点击确定，那么没问题因为地址栏改变的地址就是将要跳转到地址
   // 但是如果用户选择了取消，那么地址栏的路径已经变成了新的地址，但是页面实际还停留再之前，这就产生了bug
   // 这也就是 revertPop 这个hack的由来。因为页面的跳转可以由程序控制，但是如果操作的本身是浏览器的前进后退
  function handlePop(location) {
    if (forceNextPop) {
      forceNextPop = false;
      setState();
    } else {
      var action = 'POP';
      transitionManager.confirmTransitionTo(location, action, getUserConfirmation, function (ok) {
        if (ok) {
          setState({
            action: action,
            location: location
          });
        } else {
          // 回滚
          revertPop(location);
        }
      });
    }
  }

  // https://github.com/ReactTraining/history/issues/690
   // 这里是react-router的作者最头疼的一个地方，因为虽然用hack实现了表面上的路由拦截
   // ，但也会引起一些特殊情况下的bug。这里先说一下如何做到的假装拦截，因为本身html5 history
   // api的特性，pushState 这些操作不会引起页面的reload,所有做到拦截只需要不手懂调用setState页面不进行render即可
   // 当用户选择了取消后，再将地址栏中的路径变为当前页面的显示路径即可，这也是revertPop实现的方式
   // 这里贴出一下对这个bug的讨论：https://github.com/ReactTraining/history/issues/690
  // fromLocation 当前
  function revertPop(fromLocation) {
    // fromLocation 当前地址栏真正的地址

    var toLocation = history.location; // TODO: We could probably make this more reliable by
    // keeping a list of keys we've seen in sessionStorage.
    // Instead, we just default to 0 for keys we don't know.

    // allKeys 缓存历史堆栈数据 取出来 formLocaton 和 当天 histoty.location 维护的key在 allKeys索引中的位置
    var toIndex = allKeys.indexOf(toLocation.key);
    if (toIndex === -1) toIndex = 0;
    var fromIndex = allKeys.indexOf(fromLocation.key);
    if (fromIndex === -1) fromIndex = 0;

    // 两者进行相减的值就是go操作需要回退或者前进的次数
    var delta = toIndex - fromIndex;

    // 如果delta不为0 则进行过地址栏的变更。 浏览器历史记录重定向到当前页面的路径
    if (delta) {
      // 将forceNextPop设置为true
      // 更改地址栏的路径，又会触发handlePop 方法，此时由于forceNextPop已经为true则会执行后面的
      // setState方法，对当前页面进行rerender，注意setState是没有传递参数的，这样history当中的
      // location对象依然是之前页面存在的那个loaction，不会改变history的location数据
      forceNextPop = true;
      go(delta);
    }
  }

  // 初始化一个location对象
  var initialLocation = getDOMLocation(getHistoryState());
  var allKeys = [initialLocation.key]; // Public interface


  function createHref(location) {
    return basename + createPath(location);
  }

  function push(path, state) {
    // path 可以是字符串也可以是对象，当path传递是对象，包含state并且第二个参数state也存在，会扔出警告，第二个state将会被忽略掉
    process.env.NODE_ENV !== "production" ? warning(!(typeof path === 'object' && path.state !== undefined && state !== undefined), 'You should avoid providing a 2nd state argument to push when the 1st ' + 'argument is a location-like object that already has state; it is ignored') : void 0;
    var action = 'PUSH';
    // 返回一个对象包含 pathname,search,hash,state,key
    var location = createLocation(path, state, createKey(), history.location);
    // 路由的切换，最后一个参数为回调函数，只有返回true的时候才会进行路由的切换
    transitionManager.confirmTransitionTo(location, action, getUserConfirmation, function (ok) {
      if (!ok) return;
      var href = createHref(location); // 拼接basename后的完整路径
      var key = location.key,          // 随机生成key值
          state = location.state;      // 获取新的location中的key和state

      // 当可以使用原生的html5 history api的，调用原生的history.pushstate方法更改浏览器地址栏路径
      // 此时只是改变地址栏路径 页面并不会发生变化 需要手动setState从而rerender
      if (canUseHistory) {
        globalHistory.pushState({
          key: key,
          state: state
        }, null, href);

        // 是否开启强制刷新
        if (forceRefresh) {
          window.location.href = href;
        } else {
          // 获取上次访问key的下标
          var prevIndex = allKeys.indexOf(history.location.key);

          // 当下标存在时，返回截取到当前下标的数组key列表的一个新引用，不存在则返回一个新的空数组
          // 这样做的原因是什么？为什么不每次访问直接向allKeys列表中直接push要访问的key
          // 比如这样的一种场景， 1-2-3-4 的页面访问顺序，这时候使用go(-2) 回退到2的页面，假如在2
          // 的页面我们选择了push进行跳转到4页面，如果只是简单的对allKeys进行push操作那么顺序就变成了
          // 1-2-3-4-4，这时候就会产生一悖论，从4页面跳转4页面，这种逻辑是不通的，所以每当push或者replace
          // 发生的时候，一定是用当前地址栏中path的key去截取allKeys中对应的访问记录，来保证不会push连续相同的页面
          var nextKeys = allKeys.slice(0, prevIndex + 1);
          nextKeys.push(location.key);
          allKeys = nextKeys;

          // 通知事件调度中心，执行相对应监听函数，重新render页面
          setState({
            action: action,
            location: location  // 新的location
          });
        }
      } else {
        process.env.NODE_ENV !== "production" ? warning(state === undefined, 'Browser history cannot push state in browsers that do not support HTML5 history') : void 0;
        window.location.href = href;
      }
    });
  }

  function replace(path, state) {
    process.env.NODE_ENV !== "production" ? warning(!(typeof path === 'object' && path.state !== undefined && state !== undefined), 'You should avoid providing a 2nd state argument to replace when the 1st ' + 'argument is a location-like object that already has state; it is ignored') : void 0;
    var action = 'REPLACE';
    var location = createLocation(path, state, createKey(), history.location);
    transitionManager.confirmTransitionTo(location, action, getUserConfirmation, function (ok) {
      if (!ok) return;
      var href = createHref(location);
      var key = location.key,
          state = location.state;

      if (canUseHistory) {
        globalHistory.replaceState({
          key: key,
          state: state
        }, null, href);

        if (forceRefresh) {
          window.location.replace(href);
        } else {
          var prevIndex = allKeys.indexOf(history.location.key);
          if (prevIndex !== -1) allKeys[prevIndex] = location.key;
          setState({
            action: action,
            location: location
          });
        }
      } else {
        process.env.NODE_ENV !== "production" ? warning(state === undefined, 'Browser history cannot replace state in browsers that do not support HTML5 history') : void 0;
        window.location.replace(href);
      }
    });
  }

  function go(n) {
    globalHistory.go(n);
  }

  function goBack() {
    go(-1);
  }

  function goForward() {
    go(1);
  }

  var listenerCount = 0;

  // 防止重复注册，只有 listenerCount === 1 && delta === 1 进行监听事件
  // 同时在window上设置popstate、pushState 监听函数
  // delta=-1 移除window对象上popState pushState 等事件
  function checkDOMListeners(delta) {
    listenerCount += delta;

    if (listenerCount === 1 && delta === 1) {
      window.addEventListener(PopStateEvent, handlePopState);
      if (needsHashChangeListener) window.addEventListener(HashChangeEvent, handleHashChange);
    } else if (listenerCount === 0) {
      window.removeEventListener(PopStateEvent, handlePopState);
      if (needsHashChangeListener) window.removeEventListener(HashChangeEvent, handleHashChange);
    }
  }

  var isBlocked = false;

  // 设置路由跳转拦截监听器,这里的block专门为prompt组件服务，开发者可以模拟对路由的拦截
  // [Remove history.block #690](https://github.com/remix-run/history/issues/690)
  function block(prompt) {
    if (prompt === void 0) {
      prompt = false;
    }

    var unblock = transitionManager.setPrompt(prompt);

    // 监听事件只会当拦截器开启时被注册，同时设置isBlock为true,防止多次注册
    if (!isBlocked) {
      checkDOMListeners(1);
      isBlocked = true;
    }

    // 返回关闭路由拦截的方法
    return function () {
      if (isBlocked) {
        isBlocked = false;
        checkDOMListeners(-1);
      }

      return unblock();
    };
  }

  // 添加自定义监听函数，并返回取消监听函数
  function listen(listener) {
    var unlisten = transitionManager.appendListener(listener); // 添加订阅者
    checkDOMListeners(1);
    return function () {
      checkDOMListeners(-1);
      unlisten();
    };
  }

  var history = {
    length: globalHistory.length,  // 存储浏览器历史堆栈中数量
    action: 'POP', // 执行的方法名
    location: initialLocation, // 保存的location对象  {pathname, search, hash, state, key }构成
    createHref: createHref, // 构成完整的浏览器路径+basename
    push: push, // 自定义push事件, 实现路由跳转
    replace: replace, //  自定义replace事件, 实现路由跳转, 替换历史记录堆栈数据
    go: go, // 调用history.go方法
    goBack: goBack, // 调用history.go方法
    goForward: goForward, // 调用history.go方法
    block: block, // 设置路由跳转拦截监听函数
    listen: listen // 添加自定义路由监听函数
  };
  return history;
}


```

## 总结
- 首先BrowserRouter通过history库创建了history对象，并且把此对象通过props形势传递Router组件
- Router组件使用hisroty中listen，注册了资深的setState方法，当路由发生发生改变的时候<出发popstate,手动push>组件就会执行setState方法，完成整个组件数的render
- history是一个对象 包含了各种操作页面的方法。 同时会用Router的props里面forceRefresh、basename、getUserComfirmation、keyLength 来生成一个初始化的location对象
- 拿到从初始化的location对象，history开始封装push,replace,go,goback等方法。对于任何地址栏上的更新，都会执行confirmTransitionTo验证，这个方法是为了支持prompt拦截器功能， 正常在拦截器关闭的情况下，每次调用push或者replace都会随机生成一个key，代表这个路径的唯一hash值，并将用户传递的state和key作为state，注意这部分state会被保存到 浏览器 中是一个长效的缓存，将拼接好的path作为传递给history的第三个参数
- 地址栏地址得到更新后，页面在不使用foreceRefrsh的情况下是不会自动更新的， 需要执行setState， 同时执行调度中心里面监听函数
- history有block方法，这个方法初衷是实现对路由跳转的拦截，我们知道浏览器的回退和前进操作按钮是无法进行拦截的， 只能做hack。为了history抽离了一个路由控制器createTransitionManager，里面维护了一个prompt开关， 每当prompt存在的时候，默认会被window.confirm拦截，如果确认拦截，则页面仍然会停留在当前页面中，但是地址栏已经更新了地址，这就产生了矛盾。需要把地址栏路径重置为之前的路径。为了实现这功能。都会在allkeys找到当前key对应的下表，以及之前页面key对应的下标。以两者下标差做一个回滚
**正是因为有了Prompt才会促使history添加key**


## 手动实现mini-router

### utils
```javascript

export interface Path {
  hash: string
  search: string
  pathname: string
}

export function parsePath(path: string) {
  const partialPath = {} as Path
  const pathname = path

  if (path) {
    const hashIndex = path.indexOf('#')
    if (hashIndex >= 0) {
      partialPath.hash = path.substr(hashIndex)
      pathname = path.substr(0, hashIndex)
    }

    const searchIndex = path.indexOf('?')
    if (searchIndex >= 0) {
      partialPath.search = path.substr(searchIndex)
      pathname = path.substr(0, searchIndex)
    }

    if (path) {
      partialPath.pathname = pathname
    }
  }

  return partialPath
}


```

### history
```javascript

import { parsePath } from './utils'

export interface History {
  push(): void
}

export type State = object | null

export type Listener = (location: Location) => void

export interface Location {
  state: State
  hash: string
  search: string
  pathname: string
}

const getLocation = (): Location => {
  const { pathname, search, hash } = window.location
  return {
    pathname,
    search,
    hash,
    state: null,
  }
}

let location = getLocation()

const getNextLocation = (to: string, state: State = null) => ({
  ...parsePath(to),
  state,
})

const listeners: Listener[] = []

const push = (to: string, state?: State) => {
  location = getNextLocation(to, state)
  window.history.pushState(state, '', to)
  listeners.forEach((fn) => fn(location))
}

// 设置监听函数
const listen = (fn: Listener) => {
  listeners.push(fn)
  return function unlisten() {
    listeners = listeners.filter((listener) => listener !== fn)
  }
}

// 处理浏览器的前进后退
window.addEventListener('popstate', () => {
  location = getLocation()
  listeners.forEach((fn) => fn(location))
})

export const history = {
  get location() {
    return location
  },
  push,
  listen,
}

```


### Router
```javascript

import { history, Location } from './history'
import React, { useState, useEffect } from 'react'

interface RouterContextProps {
  location: Location
  history: typeof history
}

export const RouterContext = React.createContext<RouterContextProps | null>(null)

export const Router: React.FC = ({ children }) => {
  const [location, setLocation] = useState(history.location)

  useEffect(() => {
    const unlisten = history.listen((newLocation) => {
      setLocation(newLocation)
    })
    return unlisten
  }, [])

  return <RouterContext.Provider value={{ history, location }}>{children}</RouterContext.Provider>
}


```

### Route
```javascript
import { ReactNode } from 'react'
import { useLocation } from './hooks'

interface RouteProps {
  path: string
  children: ReactNode
}

export const Route = ({ path, children }: RouteProps) => {
  const { pathname } = useLocation()
  const matched = path === pathname

  if (matched) {
    return children
  }
  return null
}

```


### hooks

```javascript

import React from 'react'
import { RouterContext } from './Router'

export const useHistory = () => React.useContext(RouterContext)!.history

export const useLocation = () => React.useContext(RouterContext)!.location


```

### demo

```javascript

import React from 'react'
import { Card, Button, Divider } from 'antd'
import { Router, Route, useHistory } from './customizeRouter/index'

const List = () => <h1>列表</h1>
const Detail = () => <h1>详情</h1>
const About = () => <h1>关于我</h1>

const RenderRoute = () => {
  const history = useHistory()

  const go = (path: string) => {
    const state = { name: Math.random().toString(36).substr(2, 8) }
    history.push(path, state)
  }

  return (
    <div>
      <Button type="link" size="small" onClick={() => go('/list')}>
        列表
      </Button>
      <Button type="link" size="small" onClick={() => go('/detail')}>
        详情
      </Button>
      <Button type="link" size="small" onClick={() => go('/about')}>
        关于我
      </Button>
    </div>
  )
}

export default () => (
  <Card title="自定义mini-router">
    <Router>
      <RenderRoute />

      <Divider />

      <Route path="/list">
        <List />
      </Route>

      <Route path="/detail">
        <Detail />
      </Route>

      <Route path="/about">
        <About />
      </Route>
    </Router>
  </Card>
)


```