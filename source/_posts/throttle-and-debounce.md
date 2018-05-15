---
title: 一辆debounce的电梯
date: 2017-01-16 19:33:56
tags: [code]
description: throttle与debounce广泛应用于我们的生活中
---

## 发现问题

``` html
<button @click="commit | debounce 500">提交打包任务</button>

```

``` javascript
function(){
    $.ajax({
        url:'...',
        success:function(res){
            if(res.ok){
                API.tip('提交打包任务成功，请耐性等待');
            }
        }
    })
}

```

点击事件中做了提交打包任务的ajax请求，但在测试过程中偶发返回了两次提示任务提交成功的信息，打包任务耗时耗力，心疼打包服务器三秒。

在这的debounce需要用控制吗？先深入学习下debounce的使用，顺便带上throttle。

## 概念

网上的资料很多拿电梯来举例子：

> 把电梯完成一次运送，类比为一次函数的执行和响应。假设电梯有两种运行策略 throttle 和 debounce ，超时设定为15秒，不考虑容量限制。

> throttle策略的电梯。保证如果电梯第一个人进来后，15秒后准时运送一次，不等待。如果没有人，则待机。
> debounce策略的电梯。如果电梯里有人进来，等待15秒。如果又人进来，15秒等待重新计时，直到15秒超时，开始运送。

按我的理解，原话“把电梯完成一次运送，类比为一次函数的执行和响应”应该改为“乘客进入一次电梯，触发电梯等待，关门，启动的流程，类比为一次函数的执行和响应”比较合理。

### throttle

所以，throttle策略的电梯，在第一个人进入后15秒内必定会关门的，如果你是第15秒时赶到强行进入，那么画面会太美的。

### debounce

而debounce策略的电梯会给每个乘客15秒的缓冲时间，最后一个乘客进入后15秒内还没有下一个乘客到来，那么电梯就关门，启动了（虽然大家都不会等待电梯自动超时关门的）。

### So

所以，可以这么说：日常生活中的电梯都是采取debounce策略的！


## 如何用代码呈现？

### throttle的电梯
1.电梯自带一个延时器，首位进入的乘客触发延时器，超时时间x秒后关门启动。
2.在x秒内进入的乘客不会触发延迟器。
3.每次乘客进入电梯都会触发`person`。

``` javascript
    var person = throttleEvelator();

    function throttleEvelator(){
        var work = function(){
            console.log("let's go");
        };
        var timeout = 5;
        var begin = 0;
        var now = 0;
        
        return function(){
            now = new Date().getTime()/1000;
            if (now - begin > timeout) {
                begin = now;
                setTimeout(function(){
                    work();
                },timeout*1000);
            }
        }
    }
```

### debounce的电梯
1.电梯自带一个延时器，首位进入的乘客触发延时器，超时时间x秒后关门启动。
2.在x秒内进入的乘客清除了当前的延时器并重新设置延时器。
3.每次乘客进入电梯都会触发`person`。

``` javascript
    var person = throttleEvelator();

    function throttleEvelator(){
        var work = function(){
            console.log("let's go");
        };
        var timeout = 5;
        var begin;
        var now = 0;
        var timer = null;
        
        return function(){
            now = new Date().getTime()/1000;
            begin = begin || now;
            if (now - begin < timeout) {
                begin = now;
                clearTimeout(timer);
                timer = setTimeout(function(){
                    work();
                },timeout*1000);
            }else{
                begin = now;
                timer = setTimeout(function(){
                    work();
                },timeout*1000);
            }
        }
    }

    // 也可以在setTimeout的回调里做一个flag来判断当前是否应该清除延时器或直接设置延时器

    function throttleEvelator(){
        var work = function(){
            console.log("let's go");
        };
        var timeout = 5;
        var begin;
        var now = 0;
        var timer = null;
        var flag = true;
        
        return function(){
            console.log(flag);
            // flag == true 直接设置定时器
            // flag == false 清除定时器再设置
            if (flag) {
                flag = false;
                timer = setTimeout(function(){
                    work();
                    flag = true;
                },timeout*1000);
            }else{
                clearTimeout(timer);
                timer = setTimeout(function(){
                    work();
                    flag = true;
                },timeout*1000);
            }
        }
    }
```

## 使用场景

清楚了两者的用法，再回到使用场景上来，先人为什么采用`debounce`，可能是担心手抖按n下提交多次任务吧，其实这里在第一次触发的时候把按钮置为不可用就满足使用场景了。

