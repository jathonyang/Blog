## 盒模型
border-box
content-box
## bfc
()[https://juejin.im/post/5909db2fda2f60005d2093db]
块级格式化上下文，决定了元素如何对其内容进行定位以及其他元素的关系和相互作用
格式化上下文
### 生成条件
1. 根元素，HTML元素
2. float不为none
3. overflow不为visible
4. display的值为inline-block、table-cell、table-caption
5. position的值为absolute或fixed
### 布局规则
1. 内部box会在垂直方向，一个个放置
2. box垂直方向的距离由margin决定。属于同一个bfc的两个相邻box的margin会发生重叠
  解决方案: 让两个box处于不同的bfc
3. 每个元素的margin box的左边，与包含块border box的左右相接触，浮动也如此
```html
<div class="par">
    <div class="child"></div>
     <!-- 给这两个子div加浮动，浮动的结果，如果没有清除浮动的话，父div不会将下面两个div包裹，但还是在父div的范围之内。-->
    <div class="child"></div>
</div>
```
4. bfc的区域不会与float box重叠
5. bfc是页面上的独立容器，容器里面的子元素不会影响到外面的元素
### 作用
  1. 避免外边距折叠
  2. 清除内部浮动(触发父div的BFC属性，使下面的子div都处于父div的同一个bfc区域之类)
  3. 避免文件环绕
  4. 多列布局使用bfc

## 层叠上下文
()[https://juejin.im/post/59b498acf265da066563ad5f]
1. 仅在定位元素上有效果
2. 堆叠顺序由元素的层叠上下文、层叠等级共同决定
z-index为数值并且发生生效的时候，容器会发生一种变化，会使容器内的子组件无法穿过容器本身，并且子组件的层级由父组件决定。这种变化后的容器元素我们称为层叠上下文
### 层叠顺序规则
1. 层叠上下文background/border
2. 负z-index
3. block块级元素
4. float元素
5. inline-block行内元素
6. z-index: auto; z-index: 0;
7. 正z-index

## CSS动画
### transition
transition-delay: 延迟多久开始动画
transition-duration: 过度动画的一个持续时间
transition-property: 执行动画对应的属性
transition-timing-function: 动画变化轨迹的计算方式
transition: property duration timing-function delay
```css
transition-property: all, border-radius, opacity;
transition-duration: 1s, 2s, 3s;
```
### animation
animation-name: @keyframes的名字
animation-delay: 动画延长时间
animation-duration: 动画持续时间
animation-direction: 方向控制
  normal: 动画结束后，动画重置到起点重新开始
  alternate: 正向交替
  reverse: 反向运行
  alternate-reverse: 反向交替
animation-fill-mode: css属性如何应用到元素上
  none: 什么
  forwards: 最后一帧的
  backwards: 第一帧
animation-timing-function: 变化轨迹函数
animation-iteration-count: 动画循环次数
animation-play-state: 动画是否执行
```css
animation-name: red, green, blue;
animation-duration: 5s, 4s, 3s;
```
## opacity:0、visibility: hidden、display: none
1. display: none 不占空间，不能点击
2. visibility: hidden 占据空间，不能点击
3. opacity: 0(占据空间，可以点击)
## requestAnimationFrame
requestAnimationFrame应运而生，其作用就是让浏览器流畅的执行动画效果。可以将其理解为专门用来实现动画效果的api，通过这个api,可以告诉浏览器某个JavaScript代码要执行动画，浏览器收到通知后，则会运行这些代码的时候进行优化，实现流畅的效果，而不再需要开发人员烦心刷新频率的问题了
## flex
flex: auto
flex: flex-grow flex-shrink flex-basis 1 1 auto
## 清除浮动
```css
.float:after {
  content: '.';
  clear: both;
  display: block;
  height: 0;
  overflow: hidden;
  visibility:hidden;
}
```
## 设置宽高比
```css
width: 50%;
height: 0;
padding-bottom: 30%;
```

## css单位
1. px
2. % 父元素宽度的比例
3. em 相对单位
  相对父元素的字体大小
4. rem是相对于根元素的html的font-size来计算的
5. vw: 视窗的宽度计算的，vh是相对视窗的高度来进行计算的

## 看不见的空隙是怎么导致的
行框的排列会受到中间空白的影响，因为空格也属于字符，这些空白也会应用样式，占据空间，所以会有间隔，
把字符大小设置0

## 伪类和伪元素
伪元素: 在内容元素的前后插入额外的元素或样式，但是这些元素实际上不在文档中生成。他们只在外部显示可见，
但不会在文档的源代码中找到他们
伪类: 将特殊的效果添加到特定

设置浮动后display会自动变成block
## 解决1px问题
```css
.parent {
  position: relative;
}
.parent::after {
  content: '.';
  display: block;
  position: absolute;
  left: 0;
  bottom: 0;
  height: 1px;
  width: 100%;
  background-color: #000;
  transform: scaleY(0.5)
}
```
## 如何修改才能让图片宽度为300px
```html
<img src="1.jpg" style="width: 480px !important">
<style>
img {
  max-width: 300px;
}
</style>
```

## css选择器
1. !important
2. 内联选择器
3. ID选择器
4. 类选择器/属性选择器/伪类选择器
5. 元素选择器/关系选择器/伪元素选择器
6. 通配符选择器

## margin设置百分比
margin设置百分比是相对于父元素的宽度
padding设置百分比
