---
title: canvas星空
date: 2016-11-06 09:54:49
tags: [canvas,animation]
description: 用canvas为星空添加一些效果
photos: [http://7xkyc9.com1.z0.glb.clouddn.com/sky.gif]
---

效果就是这样：

![desc][1]

具体步骤如下

## 1.绘制背景

一件事，将天空铺满

```
// html代码中需要有canvas节点 
var canvas = document.getElementById('canvas');
var ctx = canvas.getContext('2d');

var width = window.innerWidth;
var height = window.innerHeight;
    
canvas.height = height;
canvas.width = width;

ctx.fillStyle = "#000";
ctx.rect(0, 0, width, height);
ctx.fill();
```

## 2.添加星星

可根据个人喜好设置，我这里的星星是这样的：
大小有别，颜色不一，位置和移动方向随机，速度值一致

```
<!-- 生成随机颜色 -->
function genColor() {
    function random() {
        return Math.round(Math.random() * 255);
    };
    return 'rgb(' + random() + ',' + random() + ',' + random() + ')';
}

<!-- 生成星星 -->
function genStars(num) {
	var stars = [];
    var star = {};
    for (var i = 0; i < num; i++) {
        star = {};
        star.id = i;
        // 设置初始位置
        star.posX = Math.random() * width;
        star.posY = Math.random() * height;
        // 设置速度,可正可负
        star.flyX = (Math.random() - 0.5);
        star.flyY = (Math.random() - 0.5);
        // 设置颜色
        star.color = genColor();
        // 设置半径
        star.r = Math.random() * 3;
        stars.push(star);
    }
    return stars;
}

<!-- 绘制星星 -->
function drawStars(){
    //生成1000个吧
	var stars = genStars(1000);
	for (var i = 0, length = stars.length; i < length; i++) {
        var star = stars[i];
        ctx.beginPath();
        ctx.fillStyle = star.color;
        ctx.arc(star.posX, star.posY, star.r, 0, Math.PI * 2, true);
        ctx.fill();
	}
}

drawStars();

```
到此一副静态的星空图已经跃然纸上，不，屏上。
* 然后引入`requestAnimationFrame()`，顾名思义请求动画的每一帧，所以回调函数中执行的都是绘制静态图的动作，即这里的`drawStars()`。

在使用动画前,考虑如何改造`drawStars()`
1. 每次绘制需要清除上次绘制图案，清空画布重新绘制的消耗较于只清除星星相对较小。
2. 使星星移动，需要在上次的位置上添加位移值，并记录移动后位置。
3. 星星如果到了画布的边界，需要使他返回，简单点就是速度方向相反就行。

改造后如下：
``` 
    var stars = genStars(1000);
    function drawStars() {
        // 清除画布并重新绘制天空
        ctx.clearRect(0, 0, width, height);
        ctx.fillStyle = "#000";
        ctx.rect(0, 0, width, height);
        ctx.fill();
        for (var i = 0, length = stars.length; i < length; i++) {
            var star = stars[i];
            // 出界后折返
            if (star.posX < 0 || star.posX > width) {
                star.flyX = -star.flyX;
            };
            if (star.posY < 0 || star.posY > height) {
                star.flyY = -star.flyY;
            };
            // 绘制星星
            ctx.beginPath();
            ctx.fillStyle = star.color;
            // 添加位移
            star.posX += star.flyX;
            star.posY += star.flyY;
            ctx.arc(star.posX, star.posY, star.r, 0, Math.PI * 2, true);
            ctx.fill();
        };
        // 继续请求下一帧
        requestAnimationFrame(drawStars);
    }
```

此时繁星移动的动画完成
![desc][2]

## 3.增加效果

为实现多种效果，获取鼠标的坐标位置是前提条件，将画布上的鼠标位置做记录，并监听鼠标移动事件，更新画布上的鼠标焦点坐标。

```
// 初始值
var mouse = {
    posX: -20000,
    posY: -20000
};
// 监听鼠标移动事件，更新画布上鼠标坐标
canvas.onmousemove = function(e) {
    mouse.posX = e.clientX;
    mouse.posY = e.clientY;
}
```

### 3.1 放射状

逻辑：以鼠标坐标为圆心，半径为R (distance)的范围内的星星与鼠标相连
```
    // 用以获取两点间距离的工具函数
    function getDistance(p1, p2) {
        return Math.sqrt(Math.pow(Math.abs(p1.posX - p2.posX), 2) + Math.pow(Math.abs(p1.posY - p2.posY), 2));
    }

    //在遍历stars时添加
    if (getDistance(mouse, star) < distance) {
        ctx.beginPath();
        ctx.moveTo(mouse.posX, mouse.posY);
        ctx.lineTo(star.posX, star.posY);
        ctx.strokeStyle = star.color;
        ctx.stroke();
        ctx.closePath();
    }
```

### 3.2 网状

逻辑：以鼠标坐标为圆心，半径为R (distance)的范围内的星星与该范围内其他星星并且距离不超过R (distance)的相互连接
```
    if (getDistance(mouse, star) < distance) {
        var _star = {};
        for (var j = 0, length = stars.length; j < length; j++) {
            _star = stars[j];
            // 链接的条件
            if (getDistance(_star, mouse) < distance && getDistance(star, _star) < distance && star.id != _star.id) {
                // 连线的渐变色
                var grd = ctx.createLinearGradient(_star.posX, _star.posY, star.posX, star.posY);
                grd.addColorStop("0", _star.color);
                grd.addColorStop("1", star.color);
                ctx.beginPath();
                ctx.moveTo(_star.posX, _star.posY);
                ctx.lineTo(star.posX, star.posY);
                ctx.strokeStyle = grd;
                ctx.lineWidth = 0.5;
                ctx.stroke();
                ctx.closePath();
            }
        }
    }
```

### 3.3 其他的一些效果

1.范围内的星星放大
```
if (getDistance(mouse, star) < distance) {
    ctx.fillStyle = star.color;
    ctx.arc(star.posX, star.posY, star.r * 3, 0, Math.PI * 2, true);
    ctx.fill();
}
```
2.范围内星星吸引
```
var dis = getDistance(mouse, star);
if (dis < distance) {
    star.flyX = (mouse.posX - star.posX) / dis;
    star.flyY = (mouse.posY - star.posY) / dis;
    if (dis + 50 > distance) {
        star.flyX = (Math.random() - 0.5) / 5;
        star.flyY = (Math.random() - 0.5) / 5;
    }
}
```
3.范围内星星排斥
```
var dis = getDistance(mouse, star);
if (dis < distance) {
    star.flyX = -(mouse.posX - star.posX) / dis;
    star.flyY = -(mouse.posY - star.posY) / dis;
    if (dis + 50 > distance) {
        star.flyX = (Math.random() - 0.5) / 5;
        star.flyY = (Math.random() - 0.5) / 5;
    }
} 
```
都是一些稍作改动就能实现的玩法。


具体[玩一下](../examples/canvas.html)

[1]: http://7xkyc9.com1.z0.glb.clouddn.com/sky.gif
[2]: http://7xkyc9.com1.z0.glb.clouddn.com/star.gif