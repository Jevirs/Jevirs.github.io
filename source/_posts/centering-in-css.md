---
title: 花式居中
date: 2015-10-20 19:20:43
tags: [code]
description: 各种条件下的水平居中，垂直居中
cover: /images/flex.png
---

说到css布局，居中是个难题，不同的情境有不同的解决方案，详情如下

# 水平居中

## 行内元素水平居中

设置容器`text-align:center;`对于行内元素起作用，也包括`display`为`inline-block`, `inline-table`, `inline-flex`的元素。

<p data-height="400" data-theme-id="dark" data-slug-hash="VPGpxz" data-default-tab="css,result" data-user="jevirs" data-embed-version="2" data-pen-title="VPGpxz" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/VPGpxz/">VPGpxz</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

## 块级元素水平居中

设置元素`margin:0 auto;`，元素需要设置宽度，否则元素宽度就是容器宽度。

<p data-height="400" data-theme-id="dark" data-slug-hash="WRgpBB" data-default-tab="css,result" data-user="jevirs" data-embed-version="2" data-pen-title="WRgpBB" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/WRgpBB/">WRgpBB</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>

## 多个块级元素水平居中

### 单行排列

1.将块元素`display`设置为`inline-block`,化为行内元素的思路。

2.利用`flex`布局，实现水平居中。

<p data-height="400" data-theme-id="dark" data-slug-hash="PWdpxB" data-default-tab="css,result" data-user="jevirs" data-embed-version="2" data-pen-title="PWdpxB" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/PWdpxB/">PWdpxB</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>

### 多行排列
使用多个单一块级元素水平居中即可。

# 垂直居中

## 行内元素垂直居中

### 单行元素垂直居中

1.如果容器的`padding-top`和`padding-bottom`相等，那么此行内容会垂直居中。

2.设置容器的`line-height`和`height`相等，这里容器需要为可设置高度的元素。

3.其他方法见下文多行元素垂直居中。

<p data-height="500" data-theme-id="dark" data-slug-hash="wgEJLN" data-default-tab="html,result" data-user="jevirs" data-embed-version="2" data-pen-title="wgEJLN" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/wgEJLN/">wgEJLN</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>

### 多行元素垂直居中

1.使用表格元素，并不推荐，造成dom结构冗余。

2.设置容器`display: table;`,元素`display: table-cell;vertical-align: middle;`,无需改变dom只套用表格样式。

3.使用flex布局，设置`display: flex;justify-content: center;flex-direction: column;`。

<p data-height="500" data-theme-id="dark" data-slug-hash="VPGpoQ" data-default-tab="css,result" data-user="jevirs" data-embed-version="2" data-pen-title="VPGpoQ" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/VPGpoQ/">VPGpoQ</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>

## 块级元素的垂直居中

1.已知元素高度的情况下，设置自容器相对于父容器的绝对定位，并设置`top: 50%;`，`margin-top`为负的高度的一半的。

2.未知元素的高度，思路和设置自容器相对于父容器的绝对定位相似，设置`top: 50%;`，只不过这里设置`transform: translateY(-50%)`而不是`margin-top`。

3.嫌麻烦吗？直接用flex布局吧。

<p data-height="500" data-theme-id="dark" data-slug-hash="qRMmWX" data-default-tab="css,result" data-user="jevirs" data-embed-version="2" data-pen-title="qRMmWX" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/qRMmWX/">qRMmWX</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>

> 注意到设置成绝对定位后，块级元素的宽度会自适应内容宽度，而flex则不会有这个问题。

# 水平垂直居中

1.已知元素的宽度或者高度，利用绝对定位后使用`margin`微调。

2.未知元素的宽度或者高度，利用绝对定位后使用`transform`微调。

3.还是flex。

<p data-height="500" data-theme-id="dark" data-slug-hash="NdLjKV" data-default-tab="css,result" data-user="jevirs" data-embed-version="2" data-pen-title="NdLjKV" class="codepen">See the Pen <a href="http://codepen.io/jevirs/pen/NdLjKV/">NdLjKV</a> by jervis (<a href="http://codepen.io/jevirs">@jevirs</a>) on <a href="http://codepen.io">CodePen</a>.</p>

# 总结

相信你看得出来flex布局代码量少，逻辑清晰，简单粗暴，以后要尽量使用flex。