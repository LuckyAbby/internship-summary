## 记录下来期间踩的 CSS 的坑

### 1. 盒模型
css 的盒模型主要有两种，分为 w3c 的标准模型以及 IE 的传统模型。 它们都是对content、padding、border以及margin的计算，但是主要的区别在于计算的方式不一样

##### 1. w3c 标准盒模型
```
元素真实的内部 width = content width + padding + border
元素真实的内部 height = content height + padding + border
```

##### 2. IE 传统盒模型
```
元素真实的 width = content width
元素真实的 height = content height
```
用一个例子来表示：

```
  <div id="boxContent"></div>
  <div id="boxBox"></div>
```
样式：

```
div {
  width:200px;
  height:200px;
  padding:10px;
  margin: 20px;
  border:5px solid black;
  background-color: #e3e3e3;
}
#boxContent {
  box-sizing: content-box;
  -moz-box-sizing: content-box; 
  -webkit-box-sizing: content-box; 
  -o-box-sizing: content-box;
  -ms-box-sizing: content-box;
}
#boxBox {
  box-sizing: border-box;
  -moz-box-sizing: border-box; 
  -webkit-box-sizing: border-box; 
  -o-box-sizing: border-box; 
  -ms-box-sizing: border-box; 
```
最后在浏览器中显示的结果如图：
![](http://ojzeprg7w.bkt.clouddn.com/css2.jpg)

上图很明显的说明了两种盒模型在处理块状元素实际的 width 时的差别。详细的计算过程就是：
```
# 对于 content-box
元素的真实内部 width = content width(200) + padding(10*2) + border(5*2)

# 对于 border-box
元素的真实内部 width = content width(200)
```

对于 border-box 这种特性在两栏布局的时候就不用再像之前再精打细算算好每个元素的宽度之后如果再需要修改类似于 padding 之类的值的时候一不小心就会被挤下来的情况。

比如：
```
<div id="content">
  <div id="left">left</div>;
  <div id="main">main</div>
</div>
```
css:
```
#content {
  width: 420px;
}
#left {
  width: 100px;
  float: left;
  margin-left: 20px;
  background: #e3e3e3;
}

#main {
  width: 300px;
  float: left;
  background: #999;
}
```
对于上面的两栏布局，必须计算好两个 div 的宽度以及 margin 等值加起来的总和等于最外层的容器的宽度。如果这个时候需要给左边的加上 padding 值，如果不改变左边的 div 的 width，这个时候就会发现右边的 div 被挤下来了。如果页面有很多地方需要细调的话，就会一不小心就把布局弄乱，这个时候就可以将两个 div 的 box-sizing 设置为 border-box, 这个时候不论怎么改变 padding border等值都不需要再重新计算值。


### 2.bfc

块级格式上下文，将当前的一个区域当做一个独立的盒子，盒子内部的布局不受外界的影响。

主要解决的两个问题

一个是清除浮动。浮动的元素无法撑起自己的父元素的高度，这个时候如果将父元素触发一个 bfc，就会引起父元素形成一个盒子撑起来高度了。

另一个就是解决外边距塌陷的问题。当两个父子 div 或者兄弟 div 之间发生外边距塌陷的时候只需要触发给父 div 或者兄弟 div 触发 bfc ，就能解决这个问题。

当然，bfc 还有另一些作用，比如用来做两栏布局等等，其实最后利用的原理都是盒子的布局不受外界的影响。