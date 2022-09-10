---
slug: CSS 居中方案
title: CSS 居中方案
authors: [justcodebryan]
tags: [CSS]
---

CSS 居中方案 📏

# 元素水平居中

将行内元素包裹在一个属性`display` 为`block`的父元素中, 父元素中添加

```css
.box {
  text-align: center;
}
```

块状元素解决方案

需要为父元素设定宽高

```css
.box {
  width: 200px;
  margin: 0 auto;
}
```

# 元素水平垂直居中

🌰`position`元素已知宽度, 绝对定位+`margin`反向偏移`(transform)`

```css
.wrap {
  position: relative;
  background-color: orange;
  width: 300px;
  height: 300px;
}
.example2 {
  background-color: red;
  width: 100px;
  height: 100px;
  position: absolute;
  left: 50%;
  top: 50%;
  margin: -50px 0 0 -50px;
  /* 第二种: 将margin换成transform, 如下 */
  /* transform: translate(-50%, -50%); */
}
```

🌰flex 布局

```css
.wrap {
  background-color: #ff8c00;
  width: 200px;
  height: 200px;
  display: flex;
  justify-content: center; /* 使子项目水平居中 */
  align-items: center; /* 使子项目垂直居中 */
}
.example3 {
  background-color: #f00;
  width: 100px;
  height: 100px;
}
```

还可以使用 table-cell 布局, 但是因为该布局对于资源的消耗过大, 现在基本没有使用, 不必了解

🌰 绝对布局

```html
<div class="wrap">
  <div class="example3">居中显示</div>
</div>
```

```css
.wrap {
  position: relative;
  background-color: orange;
  width: 200px;
  height: 200px;
}
.example3 {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: red;
  width: 100px;
  height: 100px;
  margin: auto;
}
```

🌰 给子元素相对定位, 在通过`translateY()`得到垂直居中

```css
.wrap {
  position: relative;
  background-color: orange;
  width: 200px;
  height: 200px;
}
.example3 {
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  background-color: red;
  width: 100px;
  height: 100px;
  margin: 0 auto;
}
```

🌰 利用`inline-block`的`vertical-align: middle`去对齐`after`伪元素

居中块的尺寸可以做包裹性、自适应内容，兼容性也相当好。缺点是水平居中需要考虑 inline-block 间隔中的留白(代码换行符遗留问题)

```css
.wrap {
  text-align: center;
  overflow: auto;
  width: 200px;
  height: 200px;
  background-color: orange;
}
.example3 {
  display: inline-block;
  background-color: red;
  vertical-align: middle;
  width: 100px;
  height: 100px;
}
.wrap:after {
  content: "";
  display: inline-block;
  vertical-align: middle;
  height: 100%;
  margin-left: -0.25em;
  /* To offset spacing. May vary by font */
}
```

🌰`display: flex-box`

能解决各种排列布局问题

```css
.wrap {
  display: -webkit-flex;
  display: -moz-box;
  display: -ms-flexbox;
  display: -webkit-box;
  display: flex;
  -webkit-box-align: center;
  -moz-box-align: center;
  -ms-flex-align: center;
  -webkit-align-items: center;
  align-items: center;
  -webkit-box-pack: center;
  -moz-box-pack: center;
  -ms-flex-pack: center;
  -webkit-justify-content: center;
  justify-content: center;
  width: 200px;
  height: 200px;
  background-color: orange;
}
.example3 {
  width: 100px;
  height: 100px;
  background-color: red;
}
```
