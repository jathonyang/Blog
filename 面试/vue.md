## 为什么列表组件中要写key
使用key，在patch过程中使用map效率比遍历快，主要是用于提升diff效率
## 父子组件生命周期钩子执行顺序
### 加载渲染过程
1. 父组件:beforeCreate => created => beforeMount
2. 子组件: beforeCreate => created => beforeMount => mounted
3. 父组件: mounted
### 子组件更新过程
父beforeUpdate => 子beforeUpdate => 子updated => 父updated
### 父组件更新过程
父beforeUpdate => 父updated
### 销毁过程
父beforeDestroy => 子beforeDestroy => 子destroyed => 父destroyed


深挖Vue源码

## Vue生命周期
beforeCreate => created => beforeMount => mounted => beforeDestroy => destroy
beforeUpdate => updated

## computed和watch
计算属性: 实质就是将变量的get属性重写为你的定义函数，也就是说实现了数据劫持那一步，无所谓data还是props，都可以作为计算属性函数的依赖值
属性监听，将变量丢进观察者收集器中，变化可以通知到
## computed属性
遍历每一个computed属性，创建对应的watcher。但是创建的时候并不会进行马上求值，需要触发get操作的时候才会进行getter操作，这时候进行依赖收集，和在所依赖数据的订阅者加入当前的watcher。当data中的数据值发生改变时，就会通知订阅者列表中所有的订阅者进行更新，从而触发computed重新计算
## nextTick原理
同步修改状态，异步操作DOM
Vue异步执行DOM更新，只要观察到数据变化，Vue将开始一个队列，并缓冲在同一个事件循环中发生的所有数据改变。如果同一个watcher多次触发，只会被推入到队列中一次。
js的运行机制:
js是单线程的，意思是同一时间只能做一件事情，是基于事件轮询的
1. 所有同步任务都在主线程上执行，形成一个执行栈
2. 主线程外，存在一个任务队列，只要异步任务有了运行结果，就在任务队列之中放置一个事件
3. 一旦执行栈中所有的同步任务执行完成，就会读取任务队列，看看里边有哪些事件。哪些对应的异步任务，于是结束等待状态，进入执行栈，开始执行
4. 主线程不断重复第三步
主线程的执行过程就是一个tick，所有异步结果通过任务队列来调度。任务队列中主要有两大类，macroTask和microTask。这两类task会进入任务队列。常见的macroTask有setTimeout、MessageChannel、postMessage、setImmediate。常见的microTask有MutationObserver和Promise.then

## vue如何解决白屏，有什么问题导致的
单页面页面的html是靠js生成的，首屏加载需要很大的js文件，所以当网速差的时候会产生一定程度的白屏
1. 优化webpack打包体积，code-split按需加载
2. 服务端渲染，在服务端事先拼接好首页所需的html
3. 首页加loading或骨架屏
## Vue在v-for时给每项元素绑定事件需要使用事件代理吗
使用事件代理的好处：
1. 把事件处理程序代理到父节点，减少内存占用率
2. 动态生成子节点能自动绑定事件处理程序到父节点
用v-for绑定事件时，会让节点指向同一个事件处理程序
## 例子
```js
var a = {},
b = '123',
c= 123
a[b] = 'b';
a[c] = 'c';
console.log(a[b])
// c
var a = {}, b = Symbol('123'), c = Symbol('123');
a[b] = 'b';
c[c] = 'c';
console.log(a[b]);
// b
var a = {}, b = {key: '123'}, c = {key: '456'}
a[b] = 'b';
a[c] = 'c';
console.log(a[b])
// c
```
对象的键名只能是字符串和Symbol类型
其他类型的键名会被转换为字符串类型
对象转字符串会调用toString方法
## webpack热更新原理，如何做到在不刷新浏览器的前提下更新页面
1. 当修改了一个或者多个文件
2. 文件系统接收更改并通知webpack
3. webpack重新编译构建一个或者多个模块，并通知HMR服务器进行更新
4. HMR Server使用webSocket通知HMR runtime需要更新，HMR runtime通过http请求jsonp
5. HMR runtime替换更新中的模块，如果确定这些模块无法更新，则触发整个页面刷新

## vue-router 
https://juejin.im/post/5ac61da66fb9a028c71eae1b

## 组件之间通信的方式
1. props/$emit
2. $emit/$on
3. vuex
4. $attrs/$listeners
5. provide/inject
6. $parent/$children

v-for为什么不能和v-if同时使用
当vue处理指令的时候，v-for比v-if具有更高的优先级，因此我们只渲染很小一部分，也需要遍历整个列表

## 组建中data为什么是一个函数
组件是用来复用的，且js里边对象是引用关系，如果组件中data是一个对象，那么这样作用域没有隔离，子组件的data属性值会相互影响

## vuex
State: 应用状态的数据结构
Getter: 允许组件从Store中获取数据，mapGetters辅助函数仅仅是将store中的getter映射到局部计算属性
Mutation: 唯一更改store中状态的方法，且必须是同步函数
Action: 用于提交Mutation，而不是直接变更状态，可以包含任意异步操作
Module: 允许将单一的Store拆分为多个Store并统一保存在单一的状态树中

## vue-router的路由模式
hash、history、abstract

## 虚拟DOM实现原理
主要包含下面三个部分:
1. 用js对象模拟真实DOM树，对真实DOM进行抽象
2. diff算法 - 比较两颗DOM树的差异
3. patch算法 - 将两个虚拟DOM对象的差异应用到真正的DOM树

## 代码层面的优化
长列表性能优化
图片资源懒加载
路由懒加载
第三方插件的按需引入
优化无限列表性能
服务端渲染SSR


## 使用vue计算属性和方法调用的区别
结果都是一样的，但是computed会有一个缓存的效果

## 完整的导航解析流程
1. 导航被触发
2. 在是活的组件里调用离开守卫
3. 调用全局的beforeEach守卫
4. 在重用的组件里调用beforeRouteUpdate守卫
5. 在路由配置里调用beforeEnter
6. 解析异步路由组件
7. 在激活的组件里调用beforeRouteEnter
8. 调用全局的beforeResolve守卫
9. 导航被确认
10. 调用全局的afterEach钩子
11. 触发DOM更新
12. 在创建好的实例调用beforeRouteEnter守卫传给next的回调函数

### vuex的核心概念
1. vuex的状态存储是响应式的。当Vue组件从store中读取状态的时候，若store的状态发生改变，那么相应的组件一会相应的得到更新
2. 不能直接更改store的状态，改变store的状态的唯一途径是显式的提交mutation

一个指令定义对象可以提供以下几个钩子函数
bind: 只调用一次，指令第一次绑定到元素时调用
inserted： 被绑定元素插入到父节点调用
update: 所在组件的vnode更新时调用
componentUpdated： 指令所在组件的vnode及其子vnode全部更新后嗲用
unbind： 只调用一次，指令与函数解绑时调用

### 为什么只有一个根元素
从效率上讲，如果有个多个根，那就会产生多个入口，从效率上来说不方便
如果一棵树有多个根，其实是可以优化的

### 高阶组件
```js
function WithConsole(WrappedComponent) {
  return {
    mounted() {
      console.log('I have already mounted')
    },
    props: WrappedComponent.props,
    render(h) {
      return h(WrappedComponent, {
        on: this.$listeners,
        props: this.$props,
        scopedSlots: this.$scopedSlots,
        attrs: this.$attrs,
      }, slots)
    }
  },

}
```

### vue-router如何配置404
放在最底部
{
  path: '*',
  redirect: '/404'
}

## vue-router有哪些钩子函数
全局前置: router.beforeEach
全局解析: router.beforeResolve
全局后置: router.afterEach
路由独享的守卫: beforeEnter
组件内的守卫: beforeRouteEnter、 beforeRouteUpdate、beforeRouteLeave

restful api的理解