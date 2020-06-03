# css面试宝典

## 简介说明：该宝典是整理各位大佬的面试题目及自己遇到的问题，不定时更新。

## 整合大佬的面试题及自己整理的面试，排名不分先后，感激各位大佬

**掘金：** 【**wangly19**、**sunshine小小倩**、**木易杨说、**  】



### 1、实现一个左边固定,右边自适应的DIV布局

```html
<div class="container">
  <div class="left">left固定宽度200px</div>
  <div class="right">可变宽度rightrightrightrightrightrightrightrightri</div>
</div>
```



### （1）常用写法 flex 布局

```css
.container2 {
  display: flex;
  /*设为伸缩容器*/
}
.left2 {
  width: 200px;
}
.right2 {
  flex: 1;
  /*这里设置为占比1，填充满剩余空间*/
}
```

### （2）利用bfc

```css
.left1 {
  width: 200px;
  float: left;
}

.right1 {
  margin-left: 200px;
}
```

#### （3）左浮动+margin-left  【少用 】

```css

.left {
  width: 200px;
  float: left;
}

.right {
  overflow: hidden;
}
```



### 2、使用两种方式实现box居中

```css
// DOM结构
<div class="container">
  <div class="box"></div>
</div>

// 方案一：flex方案

.container {
  width: 100%;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.container .box {
  background: red;
  min-width: 100px;
  min-height: 100px;
}

// 方案二：position方案
.container {
  width: 100%;
  height: 100%;
  position: relative;
}
.container .box {
  background: red;
  min-width: 100px;
  min-height: 100px;
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%, -50%);
}

```



### 3、如何给一组列表首尾之外的元素添加样式

```css
// 考察了css伪类的使用

// DOM
<ul class="list">
  <li class="item">1</li>
  <li class="item">2</li>
  <li class="item">3</li>
  <li class="item">4</li>
  <li class="item">5</li>
</ul>

// Style

.list > .item:not(:first-child):not(:last-child) {
  /* ... style */
}

```



### 4、css3 的transform几个属性

答： 常用的transform的属性有：

- 放大scale(x,y); x和y分别表示X和Y坐标方法的倍数，可以小于1；
- 移动translate(x,y)；x和y可以用数值，如100px，也可以用百分比表示平移其自身宽高的百分之多少； 
- 旋转rotate(x deg);  表示旋转多少度
- 实际的动画渲染效果与各种属性的顺序有关，transform执行的顺序为从左到右；

### 5、css3中的translate,transform,transition的区别

#### （1）translate:移动，transform的一个方法

通过 translate() 方法，元素从其当前位置移动，根据给定的 left（x 坐标） 和 top（y 坐标） 位置参数：

          用法transform: translate(50px, 100px);
    
              -ms-transform: translate(50px,100px);
    
              -webkit-transform: translate(50px,100px);
    
              -o-transform: translate(50px,100px);
    
              -moz-transform: translate(50px,100px);

#### （2）transform:变形。改变

- 旋转：rotate() 顺时针旋转给定的角度，允许负值 rotate(30deg)
- 扭曲：skew() 元素翻转给定的角度,根据给定的水平线（X 轴）和垂直线（Y 轴）参数：skew(30deg,20deg)
- 缩放：scale() 放大或缩小，根据给定的宽度（X 轴）和高度（Y 轴）参数： scale(2,4)
- 移动：translate() 平移，传进 x,y值，代表沿x轴和y轴平移的距离
-  所有的2D转换方法组合在一起： matrix()  旋转、缩放、移动以及倾斜元素

综合起来使用：transform: 30deg 1.5 30deg 20deg 100px 200px;

#### （3）transition: 允许CSS属性值在一定的时间区间内平滑的过渡

需要事件的触发，例如单击、获取焦点、失去焦点等

 transition:property duration timing-function delay;

 property:CSS的属性，例如：width height 为none时停止所有的运动，可以为transform

duration:持续时间

  timing-function:ease等

delay:延迟

例如：transition:width 2s ease 0s;



### 6、怎么实现三列布局（左侧和右侧宽度固定，中间自适应）

```html
<div class="container">
 <div class="left"> left </div>
 <div class="right"> right </div>
 <div class="center"> center </div>
</div>
```

#### （1）两侧浮动 + 中间自动撑开

```css
.left{
    width:200px;
    background:green;
    float:left;
}
.center{
    background:yellow;
}
.right{
    width:200px;
    background:blue;
    float:right;
}
```

#### （2）**绝对定位 + 中间板块不给宽度**

```css
.container{
    position:relative;
}
.left{
    width:200px;
    background:green;
    position:absolute;
    left:0;
}
.center{
    background:yellow;
    position:absolute;
    left:200px;
    right:200px;
}
.right{
    width:200px;
    background:blue;
    position:absolute;
    right:0;
}

```

#### （3）flex 布局 

​	**设置 flex 属性之后，子元素的 float , clear, vertical-align 属性会失效**

​	**左侧和右侧的宽度通过`flex-basis`设置宽度，中间自适应通过`flex-grow`**

#####       1. 没有设置宽度 即划分为四块 ，左右各占一块，中间 2块     

```css
.container{
    display:flex;
    flex-direction:row;
    align-items:center;
}
.left{
    flex:1;
    background:green;
}
.center{
    flex:2;
    background:yellow;
}
.right{
    flex:1;
    background:blue;
}
```

##### 	2.  设置左侧和右侧的盒子 flex-basis:希望显示的宽度; （注意：该属性的优先级高于 width ）

```css
.container{
    display:flex;
    flex-direction:row;
    align-items:center;
}
.left{
    flex-basis:200px;
    width:200px;
    background:green;
}
.center{
    flex:1;
    background:yellow;
}
.right{
    /* flex-basis 的优先级比 width 高 */
    flex-basis:100px;
    width:200px;
    background:blue;
}
```

#### （4）calc() 函数用于动态计算长度值 

​	**需要注意的是，运算符前后都需要保留一个空格**

```css
.container{
    display:flex;

}
.left{
    width:200px;
    background: yellow;
}
.center{
    width:calc(100% - 400px);
    background: #000;
}
.right{
    width:200px;
    background:blue;
}
```



### 7、flex: 0 1 auto 表示什么意思

**答**：

flex: 0 1 auto 

其实就是弹性盒子的默认值，表示 `flex-grow`, `flex-shrink` 和 `flex-basis` 的简写，

分别表示**放大比例**、**缩小比例**、**分配多余空间之前占据的主轴空间**。



> flex :flex-grow flex-shrink flex-basis
>
> ① flex-grow 剩余空间索取
>
> 默认值为0，不索取
>
> **例**: 父元素400，子元素A为100px，B为200px.则剩余空间为100
>
> 此时A的flex-grow 为1，B为2，
>
> 则 A = 100px + 100 * 1/3;  B = 200px + 100 * 2/3
>
> ② flex-shrink 子元素总宽度大于复制元素如何缩小
>
> 父400px,A 200px B 300px
>
> AB总宽度超出父元素100px;
>
> 如果A不减少，则flex-shrink ：0, B减少；
>
> ③ flex-basis
>
> 该属性用来设置元素的宽度，当然width也可以用来设置元素的宽度，如果设置了width和flex-basis，那么flex-basis会覆盖width值。



### 8、

​	**答：** 