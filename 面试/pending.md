1. 浏览器获取HTML文件，然后对文件进行解析，形成DOM Tree
2. 与此同时，进行CSS解析，生成style rules
3. 接着讲DOM Tree与Style Rules合成为Render Tree
4. 接着进入布局(Layout)阶段，也就是为每个节点分配一个应出现在屏幕上的确切坐标
5.随后调用GPU进行绘制，便利Render Tree的节点，并将元素呈现出来

浏览器重绘与重排的区别
重排：尺寸发生改变，需要重新排列元素
重绘：节点的几何属性发生改变或者样式发生改变
`<script>`: 阻塞HTML解析，执行结束后，HTML解析继续
`<script async>`: 和HTML解析过程并行。通常可用于页面统计分析
`<script defer>`: 脚本仅提取过程与HTML解析过程并行，脚本的执行将在HTML解析完毕后进行

tcp三次握手、http请求响应信息、关闭TCP链接


工厂模式:
```js
function createPerson(name){
  var person = new Object();
  person.name = name;
  person.getName = function() {
    return this.name
  }
  return person;
}
```
css

盒模型
bfc
层叠上下文
动画


原型
this
http
作用域
  bind call apply
异步方法
数组扁平化
模块化
缓存
web 安全
前端攻击
webpack loader plugin


算法:
1. 贪心算法
2. 分治算法
3. 动态规划
4. 回溯法
5. 分支限界
冒泡排序
选择排序
插入排序
快排
斐波那契