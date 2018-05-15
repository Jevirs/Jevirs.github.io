---
title: 事件模型实践
date: 2016-10-25 19:19:01
tags: [code]
cover: /images/jsevent/jslogo.jpg
description: 事件传递包含捕获，目标，冒泡三个阶段
---

# 浏览器事件模型

## 缺陷

1.说起浏览器的原生的事件模型，首先印入脑海的是`onclick`,`onload`等事件，然后才是`addEventListener`,`removeEventListener`,确实在平常的编码过程中接触到更多的是封装过的`on`,`off`之流。
2.在日常处理click等事件之时，会使用到阻止冒泡（`event.stopPropagation`），阻止默认行为(`event.preventDefault`)或者直接`reture false`,但却没有真正去深入了解事件的传递机制。

## 发现

今天阅读的时获取了很多新知识，在此作记录

首先是html
```
<!DOCTYPE html>
<html>
<head>
    <title>test</title>
    <style type="text/css">
    	#parent{
    		width: 100px;
    		height: 100px;
    		background-color: green;
    	}
    	#child{
    		width: 50px;
    		height: 50px;
    		background-color: blue;
    	}
    </style>
</head>

<body>
    <div id='parent'>
        <p id="child"></p>
    </div>
</body>

</html>
```
大概就这样
![desc][1]

然后添加事件,从window,document,html,body,parent,child依次添加捕获与冒泡事件的监听，并打印出事件的目标节点和当前节点。
```
<script>
    var parent = document.getElementById("parent");
    var child = document.getElementById("child");

    // 捕获阶段绑定事件
    window.addEventListener("click", function(e){
        console.log("window 捕获", e.target.nodeName, e.currentTarget.nodeName);
    }, true);

    document.addEventListener("click", function(e){
        console.log("document 捕获", e.target.nodeName, e.currentTarget.nodeName);
    }, true);

    document.documentElement.addEventListener("click", function(e){
        console.log("documentElement 捕获", e.target.nodeName, e.currentTarget.nodeName);
    }, true);

    document.body.addEventListener("click", function(e){
        console.log("body 捕获", e.target.nodeName, e.currentTarget.nodeName);
    }, true);

    parent.addEventListener("click", function(e){
        console.log("parent 捕获", e.target.nodeName, e.currentTarget.nodeName);
    }, true);

    child.addEventListener("click", function(e){
        console.log("child 捕获", e.target.nodeName, e.currentTarget.nodeName);
    }, true);

    // 冒泡阶段绑定的事件
    window.addEventListener("click", function(e){
        console.log("window 冒泡", e.target.nodeName, e.currentTarget.nodeName);
    }, false);

    document.addEventListener("click", function(e){
        console.log("document 冒泡", e.target.nodeName, e.currentTarget.nodeName);
    }, false);

    document.documentElement.addEventListener("click", function(e){
        console.log("documentElement 冒泡", e.target.nodeName, e.currentTarget.nodeName);
    }, false);

    document.body.addEventListener("click", function(e){
        console.log("body 冒泡", e.target.nodeName, e.currentTarget.nodeName);
    }, false);

    parent.addEventListener("click", function(e){
        console.log("parent 冒泡", e.target.nodeName, e.currentTarget.nodeName);
    }, false);

    child.addEventListener("click", function(e){
        console.log("child 冒泡", e.target.nodeName, e.currentTarget.nodeName);
    }, false);

</script>
```
点击蓝色的p标签，控制台信息
![result1][2]
是真的先捕获再冒泡吗？还是因为事件绑定的顺序？

交换捕获和冒泡的绑定顺序之后
![result2][3]
出现child的捕获冒泡顺序发生变化，原来这就是目标阶段了，当`target`与`currentTarget`相等之时就进入目标阶段，这时候绑定的事件是按绑定顺序依次触发的，所以在`child`上写的"捕获"，"冒泡"应该都改为"捕获"才对。

再做个验证：在所有事件绑定之前，添加
```
child.onclick = function(){
	console.log("child onclick");
}
parent.onclick = function(){
	console.log("parent onclick");
}

window.onclick = function(){
	console.log("window onclick");
}
```
点击得到以下结果：
![result3][4]

## 总结
1.事件的捕获是从上至下触发了`window`,`document`,`documentElement`, `body`,`parent`绑定的事件。
2.当来到`child`节点时，是目标阶段，事件按照绑定顺序依次触发。
3.冒泡阶段与捕获阶段相反，自下而上的传递，而一些`onclick`之类的事件也都在该阶段触发，但是为啥在这个阶段呢?看下一条。
4.IE的事件流中只存在冒泡这个阶段，所以为了跟好地兼容IE。。。至于那个只有捕获阶段的网景，有个了解就好。
5.到这里，标准的浏览器事件已经梳理地差不多了，可能在平时编码过程中的影响不是很大，对它的实现原理的了解不是多多益善吗？

  [1]: /images/jsevent/js1.png
  [2]: /images/jsevent/js2.png
  [3]: /images/jsevent/js3.png
  [4]: /images/jsevent/js4.png