## ['1', '2', '3'].map(parseInt)
返回`[1, NaN, NaN]`
## 防抖 & 节流
### 防抖
n秒内高频事件再次被触发，则重新计算事件
```js
function debounce(fn, wait = 1000, immediate) {
  let timeOutId, context, args;
  return function(...params) {
    let context = this;
    if (immediate) {
      fn.apply(context, params);
    }
    if (timeoutId) {
      clearTimeout(timeOutId)
    }
    timeOutId = setTimeout(() => {
      fn.apply(context, params);
    }, wait)
  }
}
```
### 节流
高频事件触发，但在n秒内只会执行一次
```js
function throttle (fn, wait) {
  let timeoutId = null
  return function (...params) {
    let context = this;
    if (!timeoutId) {
      timeoutId = setTimeout(() => {
        fn.apply(context, params);
        timeoutId = null;
      }, wait);
    }
  }
}
```
## 深拷贝
```js
function cloneDeep(source, hash = new WeakMap()) {
  if (!isObject(source)) return source;
  if (hash.has(source)) return hash.get(source);
  let target = Array.isArray(source) ? [...source] : { ...source };
  hash.set(source, target);
  Reflect.ownKeys(target).forEach((key) => {
    if (isObject(source[key])) {
      target[key] = cloneDeep(source[key], hash);
    } else {
      target[key] = source[key];
    }
  })
  return target;
}
```
## 扁平化数组
```js
// section 1
function flatten(arr) {
  return arr.reduce((result, item) => {
    const temp = Array.isArray(item) ? flatten(item) : item;
    result.concat(temp);
  }, [])
}
```
## 观察者模式和订阅-发布模式
简单来说就是订阅-发布模式有中间商赚差价
## 隐式转换
```js
let a = {
  i: 1,
  toString() {
    return a.i++
  }
} 
if (a == 1 && a == 2 && a == 3) {
  console.log('i')
}
```
## 箭头函数
1. 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象
2. 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用rest参数代替
3. 不可以使用yield命令，因此箭头函数不能用作generator函数
4. 不能使用new命令，因为：
 1. 没有自己的this,无法调用call, apply
 2. 没有prototype属性，而new 命令在执行时需要将构造函数的prototype赋值给新的对象的`__proto__`

## 事件委托
### 优势
1. 减少事件注册，节省内存
2. 减少DOM节点更新的操作，处理逻辑只需要在委托元素上进行
  - 新添加的li不用绑定事件
  - 删除li时，不需要进行元素和处理函数的解绑
### 劣势
1. 事件委托依赖于冒泡，对于onfocus和onblur事件不支持
2. 层次过多，冒泡过程中，可能会被某层阻止掉
### jsonp
```js
function jsonp(url, jsonpCallback, success) {
  let script = document.createElement('script');
  script.src = url;
  script.async = true;
  script.type = 'text/javascript';
  window[jsonpCallback] = function(data) {
    success && success(data)
  }
  document.body.appendChild(script)
}
jsonp(url, callbackName, function(value) {
  console.log(value)
})
```
## Load和DOMContentLoaded区别
load事件触发代表页面中的DOM、CSS、JS，图片已经加载完成
DOMContentLoaded表示HTML被完全加载和解析
## new 
```js
function create(fn, ...params) {
  const obj = Object.create(fn.prototype);
  const ret = fn.apply(obj, params);
  return ret instanceof Object ? ret : obj;
}
```
## instanceof
```js
function instanceof(left, right) {
  let prototype = right.prototype;
  left = left.__proto__;
  while(true) {
    if (left === null) {
      return false;
    }
    if (prototype === left) {
      return true;
    }
    left = left.__proto__
  }
}
```
## this
```js
function foo() {
  console.log(this.a)
}
var a = 1;
foo();
var obj = {
  a: 2,
  foo: foo
}
obj.foo();
// 以上两种情况this只依赖于调用函数前的对象
// 优先级是第二个情况大于第一个情况

// 以下情况是优先级最高的，this只会绑定在c上，不会被任何方式修改
var c = new Foo();
c.a = 3;
console.log(c.a)

```

## 执行上下文
- 全局执行上下文
- 函数执行上下文
- eval执行上下文

```js
var foo = 1
(function foo() {
    foo = 10
    console.log(foo)
}()) 
console.log(foo)
```
当js解释器遇到非匿名的立即执行函数时，会创建一个辅助的特定对象，
然后将函数名称作为整个对象的属性，因此函数内部可以访问到foo，
但是这个值是可读的，所以对它的赋值并不生效

## call、apply、bind
### bind
```js
Function.prototype.bind = function (context, ...param1) {
  const self = this;
  function Empty() {};
  function bound(...param2) {
    return self.apply(this instanceof bound ? this : context, [...param1, ...param2])
  }
  Empty.prototype = self.prototype;
  bound.prototype = new Empty();
  return bound;
}
```
### call
```js
Function.prototype.call = function(context, ...params) {
  context.__fn__ = this;
  const res = context.__fn__(...params);
  delete context.__fn__
  return res;
}
```
### apply
```js
Function.prototype.apply = function(context, ...params) {
  context.__fn__ = this;
  const res = context.__fn__(params);
  delete context.__fn__;
  return res;
}
```

## 原型
```js
function Person() {

}
console.log(Person === Person.prototype.constructor);
console.log(Object.getPrototypeOf(person) === Person.prototype);
console.log(Person.prototype.__proto__ === Object.prototype)
console.log(Object().prototype === Object.prototype)
console.log(Object.prototype.constructor === Object())
```

```js
var value = 1;
function foo() {
  console.log(value)
}
function bar() {
  var value = 2;
  foo();
}
bar();

var scope = 'global scope';
function checkScope() {
  var scope = 'local scope';
  function f() {
    console.log(scope)
    return scope;
  }
  return f();
}
checkScope();
var scope = "global scope";
function checkScope() {
  var scope = "local scope";
  function f() {
    return scope;
  }
  return f;
}
checkScope()();
```
## 变量对象
```js
function foo() {
  console.log(a)
  a = 1;
}
foo();
function bar() {
  a = 1;
  console.log(a)
}
bar();
```
## 模拟new
```js
function objectFactory(obj, ...params) {
   const obj = Object.create(fn.prototype);
    const ret = fn.apply(obj, arg);
    return ret instanceof Object ? ret : obj;
}
```
## 继承模式
### 原型链继承
```js
function Parent() {
  this.property = true;
  this.colors = ['red', 'blue', 'green']
}
Parent.prototype.getParentValue = function () {
  return this.property;
}
function Child() {
  this.childProperty = false;
}
Child.prototype = new Parent();
Child.prototype.getChildValue = function () {
  return this.childProperty
}
var instance1 = new Child();
var instance2 = new Child();
instance1.colors.push('black')
console.log(instance1.colors)
// ['red', 'blue', 'green', 'black']
console.log(instance2.colors)
// ['red', 'blue', 'green', 'black']
```
### 借用构造函数
```js
function Parent() {
  this.colors = ['red', 'blue']
}
function Child() {
  Parent.call(this)
}
```
### 组合继承
```js
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'blue']
}
Parent.prototype.sayName = function () {
  console.log(this.name)
}
function Child(name, job) {
  // 继承属性
  Parent.call(this.name)
  this.job = job;
}
Child.prototype = new Parent();
Child.prototype.constructor = Child;

```
### 寄生组合式继承
```js
function Parent(name) {
  this.name = name;
  this.colors = ['red', 'blue']
}
function Child(name, job) {
  Parent.call(this, name);
  this.job = job;
}
Child.prototype = Object.create(Parent.prototype);
Child.prototype.constructor = Child;
```
## 作用域链
在查找变量的时候，会先从当前上下文的变量对象中查找，如果没有找到，就会从父级执行上下的变量对象中查找，一直找到全局上下文的变量对象
## 作用域
在运行时代码中某些特定部分中变量、函数和对象的可访问性
## MVVM
Model-View-ViewModel 
model代表数据模型，也可在model中定义数据修改和操作的业务逻辑
View代表UI组件，负责将数据模型转换为UI展现出来
ViewModel代表View和Model的对象同步
## 接口防刷
1. 网关控制流量洪峰，对在一个时间段内出现流量异常，可以拒绝请求
2. 源ip请求个数限制，对请求来源的ip请求个数做限制
3. http请求头信息校验
4. 对用户唯一身份uid进行限制和校验
5. 验证码的方式
## await
```js
function await() {
  return new Promise(resolve => {
    setTimeout(resolve, 10 * 1000)
  })
}
async function main() {
  console.time();
  await wait();
  await wait();
  await wait();
  console.timeEnd();
}
main();
```
30s多一点
主流程的执行必须要等到await函数执行完才能陆续执行后续函数
`await wait() => promise => wait() => resolve()`
但是`wait`函数本身又写了一个`promise`。整个`promise`会`setTimeout`才能`resolve`，相当于`wait函数=>promise=>setTimeout(resolve)`
## await2
```js
function wait() {
  return new Promise((resolve) => {
    setTimeout(resolve, 10 * 1000)
  })
} 
async function main() {
  console.time();
  const x = wait();
  const y = wait();
  const z = wait();
  await x;
  await y;
  await z;
  console.timeEnd();
}
main();
```
## 如何解决大量数据渲染
1. 冻结对象
2. 异步渲染组件
3. 使用v-if最多显示上下中三屏，避免渲染大量DOM节点
4. 使用分页

## http2
### http1存在的问题
1. 对于同一个域名，浏览器最多只能同时创建6-8个tcp链接。为了解决数量限制，出现了域名分片技术，就是资源分域，把资源放在不同域名下，就可以针对不同域名创建连接并请求，已一种讨巧的方式突破限制
2. 线头阻塞：虽然实现了keep-alive，实现了多个请求公用一个tcp连接，但是第一个响应还是会阻塞后续的响应，
3. header内容多，每次请求header不会变化太多，没有响应的压缩传输优化方案
4.为了尽可能减少请求书，需要做合并文件、雪碧图、资源内联等优化工作，但是这无疑造成了单个请求内容变大延迟变高的问题
### http2的优势
1. 二进制分帧层: 帧是数据传输的最小单位，以二进制传输代替原来的明文传输
2. 多路复用：在一个tcp连接上，我们可以向对方不断发送帧，每帧的stream identifier的标明这一帧是属于哪个流，然后在对方接受时，根据stream identifier拼接每个流的所有帧组成一整块数据。把http1.1每个请求当做一个流，多个请求变成多个流，请求响应数据分成多个帧，不同流的帧交错得发送给对方，这就是http/2的多路复用
3. 服务端推送： 服务器主动推送某个请求的资源，浏览器不用发起后续请求。相对于http/1.1资源内联，客户端可以缓存推送的资源，推送资源不可不同页面共享
3. header压缩
缓存方面： http1.0主要是强缓存，http1.1加入了更多的缓存机制

## 301和302的区别
对用户来说没什么区别，都是一个旧地址跳转到一个新地址，内容指向了新地址，主要区别在搜索引擎
301: 认为旧地址不会在用到，会把地址跟内容都更新成新地址和内容
302: 搜索引擎任务跳转只是临时的，会保留原地址，抓取新的内容
## 栗子
```js
[1, 2] + [2, 1]
"1, 22,1"
```
js中所有对象基本都是先调用valueOf方法，如果不是数值，在调用toString方法
```js
"a" ++ "b"
"aNaN"
```
+ 作为一元操作符，如果操作数是字符串，则调用Number方法将该操作数转为数值，
如果操作数无法转为数值，则为NaN
## 例子
```js
var name = "Tom"
(function () {
  if (typeof name == 'undefined') {
    name = 'jack';
    console.log('GoodBye' + name)
  } else {
    console.log('Hello ' + name)
  }
})()
// Tom
// 如果这里name 使用var， 就会变量提升
// jack
```
### 例子
```js
String('11') == new String('11')
String('11') === new String('11')
```
new String调用生成的是一个对象，如果用==会调用valueOf, toString
最后进行对比，上面的new String就会先调用toString在进行对比
## 模拟localStorage和过期时间
```js
const localStorage = (function () {
  const store = {};
  const storeTimer = {};
  return {
    getItem: function (key) {
      return store[key] || null
    },
    setItem: function(key, val, time) {
      time = Number(time) || 0;
      store[key] = val.toString();
      if (time > 0) {
        this.timeOut(key, time)
      }
    },
    timeOut: function(key, time) {
      storeTimer[key] = setTimeout(() => {
        this.removeItem(key);
      }, time)
    }, 
    removeItem: function(key) {
      delete store[key]
      clearTimeout(storeTimer[key])
    },
    clear: function () {
      store = {};
      Object.keys(storeTimer).forEach((key) => {
        storeTimer[key] = null;
      })
      storeTimer = {};
    }
  }
})()
Object.defineProperty(window, 'localStorage2', {
  value: localStorage
})
```
## 例子
```js
function Foo() {
  Foo.a = function () {
    console.log(1)
  }
  this.a = function () {
    console.log(2)
  }
}
Foo.prototype.a = function() {
  console.log(3)
}
Foo.a = function() {
  console.log(4)
}
Foo.a() // 4
let obj = new Foo();
obj.a() // 2
Foo.a() // 1
```
## let var const的区别
1. var和let用于声明变量，const用于声明只读的常量
2. var声明的变量，不存在块级作用域，在全局范围内都有效，let 和const声明的，只有他的代码块中有效
3. let和const不存在像var那样的变量提升，所以var定义变量可以先使用，后声明，而let和const只可先声明，后使用
4. let和const都只能声明一次，const在初始化的时候必须赋值
## promise.all原理以及错误处理
promise.all返回resolve，但当有一个失败reject，则返回失败的消息，机器其他promise执行失败，也会返回失败
### 如何使用
Promise.all([promise1, promise2])
### 实现原理
```js
function promiseAll(promises) {
  return new Promise(function(resolve, reject) {
    if (!Array.isArray(promises) {
      return reject(new TypeError('argument must be an array'))
    })

    let countNum = 0;
    let promiseNum = promises.length;
    const resolvedValue = new Array(promiseNum);
    for (let i = 0; i < promiseNum; i++) {
      const promise = promises[i].then ? promises[i] : Promise.resolve(promises[i])
      promise
        .then((value) => {
          countNum++;
          resolvedValue[i] = value;
          if (countNum === promiseNum) {
            return resolve(resolvedValue)
          }
        })
        .catch(reason) {
          return reject(reason)
        }
    }
  })
}
```
### promise.all错误处理
如何做到一个promise.all中即使一个promise程序reject，promise.all依然能将其他数据正常返回呢
当promise捕获到error的时候，代码吃掉这个异常，返回resolve，约定特殊格式表示这个调用成功了
```js
new Promise((resolve, reject) => {
  try {
    console.log(xx.bb)
  } catch(exp) {
    resolve('error')
  }
})
```
## 设计并实现promise.race
```js
Promise._race = promises => new Promise((resolve, reject) => {
  promises.forEach((promise) => {
    promise.then(resolve, reject)
  })
})
```
## promise的finally实现原理
```js
// todo: 
Promise.prototype.finally = function(callback) {
  const P = this.constructor;
  return this.then(
    value => P.resolve(callback()).then(() => value),
    reason => P.resolve(callback()).then(() => {
      throw reason;
    });
  )
}
```
## ES6代码转换为ES5代码的实现思路是什么
把es6的代码转换为AST语法树，然后再把es6 ast转为es5 ast，再讲ast转为代码

## 箭头函数和普通函数的区别是什么
箭头函数
1. 函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象
2. 不可以使用arguments对象，该对象在函数体内不存在。如果要用，可以用rest参数代替
3. 不能使用yield命令，所以不能用作generator函数
4. 不可以使用new命令
  没有自己的this，无法调用call, apply
  没有prototype属性，而new命令在执行时需要构造函数的prototype赋值给新的对象的__proto__
## 例子
```js
var a = {n: 1}
var b = a;
a.x = a = {n: 2}
console.log(a.x)
console.log(b.x)
// undefined
// {n: 2}
// 上面等同于
b.x = a = {n: 2}
// 因为.的运算等级比=高
```
## 为什么使用图片进行埋点
1. 资源小
2. 不存在跨域问题
3. 不会阻塞页面加载
## example
```js
var obj = {
  '2': 3,
  '3': 4,
  'length': 2,
  'splice': Array.prototype.splice,
  'push': Array.prototype.push
}
obj.push(1)
obj.push(2)
console.log(obj)
// [,,1,2]
console.log(obj[2])
// 1
```
## https握手过程
1. 客户端使用https的url访问web服务器，要求与服务器建立ssl连接
2. web服务器收到客户端请求后，会将网站的证书发送一份给客户端
3. 客户端收到网站证书会检查证书是否合法，如果没有问题就随机产生一个秘钥
4. 客户端利用服务端的公钥将会话秘钥加密，并发送给服务端，服务端利用自己的私钥解密出会话秘钥
5. 之后服务器和客户端使用秘钥进行加密传输
## https握手过程中，客户端如何验证证书的合法性
1. 是否是信任的有效证书，浏览器内置了信任的根证书，看看web服务器上的证书是不是这些信任的根证书的或者信任根的二级证书机构颁发的，还有是否过期，是否被吊销
2. 证明对方是否持有证书的私钥
## 实现一个sleep
```js
function sleep(time) {
  return new Promise(resolve => setTimeout(resolve, reject))
}
sleep(1000)
.then(() => {

})
async function sleepAsync(fn) {
  await sleep(100)
  fn();
}
```
## 缓存
https://www.jianshu.com/p/54cc04190252
## 回流和重绘
### 浏览器的渲染机制
1. 生成DOM Tree，生成CSSOM树
渲染树只包含可见的节点
不可见的节点包括： 
  1. 不会渲染的输出的节点，如果script，meta， link
  2. 一些通过css进行隐藏的节点
2. 将DOM树和CSSOM树结合，生成render tree
3. Layout: 根据生成的渲染树，进行回流，得到节点的几何信息
4. painting: 根据渲染树以及回流得到的几何信息，得到节点的绝对像素
5. display: 把像素发送给GPU，展现在页面上
### 什么时候会发生回流重绘
1. 添加或删除可见的DOM元素
2. 元素的位置发生变化
3. 元素的尺寸发生变化
4. 内容发生变化
5. 页面一开始渲染的时候
6. 浏览器尺寸发生变化的时候
## 发生重绘
当页面中元素样式的改变并不影响他在文档流中的位置
### 浏览器的优化机制
每次重排都会造成额外的计算消耗，因此大多数浏览器会通过队列滑修改并批量执行来优化重排过程。浏览器会把修改操作放入到队列中，直到过了一段时候或者操作达到了一个阈值，才会清空队列。当你获取布局信息的操作的时候，会强制队列刷新
### 减少回流和重绘
1. 使用cssText
2. 修改CSS的class
### 批量修改DOM
1. 使元素脱离文档流
2. 对其多次修改
3. 将元素待会到文档中
有三种方式可以让DOM脱离文档流
1. 隐藏元素，应用修改，重新显示
2. 使用文档片段，在当前DOM之外构建一个子树，再把它拷回文档
3. 把原始元素拷贝到一个脱离文档的节点中，修改节点后，在替换原始节点
## 避免触发同步布局事件
如下:
```js
function initP() {
  for (let i = 0; i < pas.length; i++) {
    pas[i].style.width = box.offsetWidth + 'px'
  }
}
// 这里可以先对box.offsetWidth进行缓存
width = box.offsetWidth;
```
### 对于复杂动画效果，使用绝对定位让其脱离文档流
### 使用GPU加速
使用硬件加速，可以让transform、opacity、filters这些动画不会引起回流重绘

## 判断是不是数组的情况
1. Object.prototype.toString.call();
2. instanceof
3. Array.isArray()
4. constructor
[].constructor === Array
## es5/es6的继承除了写法还有什么区别
1. class声明会提升，但不会初始化赋值
2. class内部采用严格模式
3. class所有方法都是不可枚举的
4. class所有方法都没有原型对象prototype，所以不能使用new来嗲用
## 例子
```js
const promise = new Promise((resolve) => {
  console.log(1);
  resolve();
  console.log(2)
})
promise.then(() => {
  console.log(3)
})
console.log(4)
```
## set、map、weakSet、weakMap
set:
  成员唯一、无序且不重复
  [value, value]，没有键值
  可以遍历
weakSet:
  成员都是对象
  成员是弱引用，可以被垃圾回收机制回收
  不能遍历，方法有add、delete、has
Map:
  本质上是键值对的集合
  可以遍历
WeakMap
  只接受对象作为键名，不接受其他类型的值
  键名是弱引用，键值可以是任意的
  不能遍历

## 模块化
1. AMD: 使用requireJS来编写模块化，依赖必须提前声明好
2. CMD： 使用seaJS来编写模块化
3. CommonJS: node自带的模块化
4. UMD：兼容AMD, CommonJS模块化语法

## 骨架屏实现思路
封装多个不同大小的模块
根据页面需求配置组合模块
请求数据并处理成功后替换

## setTimeout、promise、async/await
setTimeout的回调函数放到宏任务队列里，等到执行栈清空以后执行；promise.then里的回调函数会放到微任务队列中;
async函数表示函数里边可能会有异步方法，await后面跟一个表达式，async方法执行时，遇到await会立即执行表达式，然后把表达式后面的代码放到微任务队列中

## '1'.toString()为什么可以调用
实际做了这些事情
```js
var s = new String('1')
s.toString();
s = null
```
## 实现instanceof的功能
```js
function myInstanceOf(left, right) {
  if (typeof left !== 'object' || left === null) return false;
  let proto = Object.getPrototypeOf(left);
  while(true) {
    if (proto == null) return false;
    if (proto == right.prototype) return false;
    proto = Object.getPrototypeof(proto)
  }
}
```
## 对象转原始类型是根据什么流程运行的
1. 如果有Symbol.toPrimitive()方法，优先调用在返回
2. 调用valueOf(),如果转换为原始类型，则返回
3. 调用toString(), 如果转换为原始类型
4. 如果都没有返回原始类型，会报错
## 闭包
闭包是有权访问另外一个函数作用域中的变量的函数
## 例子
```js
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, 0)
}
```
修改
1. 利用IIFE
```js
for (var i = 1; i <= 5; i++) {
  (function(j) {
    setTimeout(function timer() {
      console.log(j)
    }, 0)
  })(i)
}
```
2. 给定时器传入第三个参数
```js
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer(j) {
    console.log(j)
  }, 0, i)
}
```
## 原型链的理解
每当定义一个函数数据类型的时候，都会天生自带一个prototype属性，这个属性指向函数的原型对象
js对象通过prototype指向父类对象，直到Object对象

## load和DOMContentLoaded
解析HTML结构
加载外部脚本和样式表文件
解析并执行脚本代码
构造HTML DOM模型 // DOMContentLoaded
加载图片等外部文件
页面加载完成 // load

## eventBus实现
```js
class EventBus {
  constructor() {
    this._events = new WeakMap();
  }
  on(type, fn) {
    const handler = this._events.get(type);
    const handlers = handler || [];
    this._events.set(type, handlers.push(fn))
  }
  emit(type, ...params) {
    const handlers = this._events.get(type);
    for (let i = 0; i < handlers.length; i++) {
      handler[i].apply(this, params);
    }
  }
  off(type, fn) {
    const handlers = this._events.get(type);
    const index = handlers.findIndex((val) => val === fn)
    if (index > -1) {
      handler.splice(index , 1);
    }
  }
}
```

## Promise
```js
function myPromise(constructor){
    let self=this;
    self.status="pending" //定义状态改变前的初始状态
    self.value=undefined;//定义状态为resolved的时候的状态
    self.reason=undefined;//定义状态为rejected的时候的状态
    function resolve(value){
        //两个==="pending"，保证了状态的改变是不可逆的
       if(self.status==="pending"){
          self.value=value;
          self.status="resolved";
       }
    }
    function reject(reason){
        //两个==="pending"，保证了状态的改变是不可逆的
       if(self.status==="pending"){
          self.reason=reason;
          self.status="rejected";
       }
    }
    //捕获构造异常
    try{
       constructor(resolve,reject);
    }catch(e){
       reject(e);
    }
}
myPromise.prototype.then=function(onFullfilled,onRejected){
   let self=this;
   switch(self.status){
      case "resolved":
        onFullfilled(self.value);
        break;
      case "rejected":
        onRejected(self.reason);
        break;
      default:       
   }
}
```
## 模块化
require支持动态引入，import不支持
require是同步导入，import属于异步导入
require是值拷贝，导出值变化不会影响导入值;import指向的是
内存地址，导入至会随导出值而变化
## SSR的原理
app.js作为客户端与服务端的公用入口，导出Vue根实例，供客户端entry和服务端entry使用。客户端entry主要作用挂载到DOM上，服务端entry除了创建和返回实例，还进行路由匹配与数据预获取
webpack为客户端打包一个client bundle,为服务端打包一个server bundle
服务端接收请求时，会根据url，加载相应组件，获取和解析异步数据，创建一个读取Server Bundle的BundleRenderer, 然后生成html发送给客户端
客户端混合，客户端收到服务端传来的DOM和自己生成的DOM进行对比，把不相同的DOM激活，能够相应后续辩护，为确保混合成功，客户端与服务端需要共享一套数据。在服务端之前获取数据，填充到store里，这样在客户端挂在到DOM之前，可以直接从store中获取

## 执行上下文
变量环境: 保存了变量提升的内容
词法环境

hook

## CORS
浏览器会自动向请求头添加origin字段，表明当前请求来源
服务器需要设置响应头的`Access-Control-Allow-Methods`、`Access-Control-Allow-Headers`、`Access-Control-Allow-Origin`
指定允许的方法、头部、源信息
请求分为简单请求和非简单请求，非简单请求会进行一次option方法进行预检，看看是否允许当前跨域请求
简单请求:
请求方法是三种之一:
head、get、post
http的请求头不超出以下几种字段:
Accept、Accept-Language、 Content-Language、Last-Event-ID、Content-Type


运行es6的代码相对转换后的es5有以下优点:
1. 对于一样的逻辑，用es6实现的代码量比es5更少
2. js引擎对es6的语法进行性能优化

## mvvm
model-view-viewModel 促进了前端开发与后端业务逻辑的分离，核心是ViewModel层，负责转换model中的数据对象来让数据变得更容易管理和使用，该层向上与视图层进行双向数据绑定，向下与Model层通过接口请求进行数据交互

## v-model的原理
text、textarea使用value属性和input事件
checkbox、radio使用checked属性和change事件
select使用value和change

## 非工程化项目初始化页面闪动问题
页面加载的时候闪烁花括号，v-cloak指令和css规则如[v-cloak]{display: none}

## 同源
协议/域名/端口

## pwa
### 能干什么
1. 消息推送
2. 主屏ICON，全屏访问
3. 离线存储

## WebAssembly
### js在在引擎中会经历什么
1. js文件会被下载下来
2. 然后进入解析，解析包括词法分析，语法分析把代码转换为AST
3. 然后根据抽象语法树， 字节码编译器会生成引擎能够直接阅读和执行的字节码
4. 字节码进入翻译器，将字节码一行一行的翻译为效率十分高的Machine Code
在项目运行的过程中，引擎会对执行次数较多的function进行优化，引擎会将其代码编译成Machine Code
后打包到顶部的JIT Compiler。下次在执行这个function，就会执行编译好的Machine Code。但是js的动态变量，上一秒是Array,下一秒就变成了Object。那么上一次引擎做的优化，就失去了作用，此时需要再一次优化

wasm并不是一种编程语言，而是一种新的字节码格式。和js需要执行不同的是，wasm字节码和底层机器码很相似可快速装载运行，因此性能相对于js解释执行有很大的提升

## for...of 和for...in
1. 推荐在循环对象属性的时候，使用for...in在遍历数组的时候使用for...of
2. for...in循环出的是key, for...of循环出的是value
3. for...of是es6新引入的特性
4. for...of不能循环普通的对象，需要通过和Object.keys()配合使用


一个进程有多个线程

## 多个关联的异步请求
```js
const request = url => {
  return new Promise((resolve, reject) => {
    $.get(url, data => {
      resolve(data)
    })
  })
}
request(url).then(data1 => {
  return request(data.url)
})
```
## Promise.prototype.then
then中的函数一定要return一个结果或者新的promise对象，才可以让之后的
then回调接受
## question
### question1
```js
const promise = new Promise((resolve, reject) => {
  console.log(1);
  resolve();
  console.log(2)
})
promise.then(() => {
  console.log(3)
})
console.log(4)
```
Promise构造函数是同步执行的，promise.then中的函数是异步执行的
