# 块级格式化上下文（BFC）

参考博文:

<https://www.zhangxinxu.com/wordpress/2015/02/css-deep-understand-flow-bfc-column-two-auto-layout/>

<http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html>



### 1.格式化上下文（Formatting context）
​	格式化上下文是CSS2.1规范中的概念。它是页面中的一块独立的渲染区域，有着自己的渲染规则，决定其子元素的定位等，以及自身与其他元素之间的关系。

​	CSS2.1中有两种格式化上下文：Block Formatting context (BFC) 和 inline Formatting context(IFC)。

### 2.BFC的规则

- BFC	区域不会与浮动（float）元素重叠
- BFC在页面中是一个独立的容器，内部的子元素不会影响到外部的元素，返回如此
- BFC其下相邻的普通子元素（不是BFC元素）的margin会发生重叠
- 计算BFC的高度时，其浮动元素也会参与计算。意思就是说：父元素为BFC时，其下如果是float的元素，父元素的高度不会塌陷，依然会被子元素的高度撑高。

### 3.如何才能生成BFC

- 根元素
- float不为none
- position: absolute / fixed
- display: inline-block / table-cell / table-caption / flex / inline-flex
- overflow不为visible

### 4.BFC有什么作用

- 解决当子元素浮动元素时，父元素高度塌陷的问题
- 解决相邻子元素margin重叠的问题
- 实现自适应的多栏布局

### 5.如何实现自适应多栏布局

```html
<style>
    .parent{
     	display: table-cell;   
    }
</style>

<body>
    <div class='parent'>
    	<div class='child'></div>
    </div>
    <div class='parent'></div>
    <div class='parent'></div>
</body>
```

注意：.child必须设置display: block / inline-block，这时.child设置width属性才起作用。