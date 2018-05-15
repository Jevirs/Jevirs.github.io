---
title: 认知flux处理数据流
date: 2016-03-23 17:17:02
tags: [React,Flux]
description: 这篇文章来自国外一位作者的，文章没有深究flux的具体实现，但对于想入门flux的你会有所收获
photos: [http://7xkyc9.com1.z0.glb.clouddn.com/flux.png]
---
这篇文章来自国外一位作者的[Flux For Stupid People](http://blog.andrewray.me/flux-for-stupid-people/)，文章没有深究flux的具体实现，但对于想入门flux的你会有所收获。

## 我应该使用Flux么
- 如果你的应用有很多动态数据需要处理，你可能需要flux。
- 如果你的应用多包含静态视图，并且很少涉及到数据的存储和更新，那么flux也许不会给你带来任何好处。

## 什么是Flux
Flux通过事件和监听来实现了单向数据流，具体请看后文。
我们在示例代码中使用了这两个库：[Flux Dispatcher](https://github.com/facebook/flux/blob/master/src/Dispatcher.js)和[microevent.js](http://notes.jetienne.com/2011/03/22/microeventjs.html)。

官方的文档过于意识流，不适合新手学习。我们在学习时也不要将Flux与一些MVC框架相比，否则你会更加迷惑。


*下面是一些Flux中提到的基本概念*：
## 1. Your Views "Dispatch" "Actions"(视图触发事件)
dispatcher 是一个基本的事件系统，他有自己的一些规则。他会广播事件，并注册回调函数。这里我们有仅一个全局的dispatcher。你应该使用FB的[Dispatcher Library](https://github.com/facebook/flux/blob/master/src/Dispatcher.js)。初始化很容易：
``` js
var AppDispatcher = new Dispatcher(); 
```
假设你有一个按钮，点击之后会向列表增加一条数据。
``` html
<button onClick={this.createNewItem}>New Item</button> 
```
点击之后，你的视图会分发一个action，其中有action的名称和增加的数据内容:
``` js
createNewItem:function(evt){
  AppDispatcher.dispatch({
      actionName: 'new-item', 
      newItem: {name:'Marco'} // 所需要增加的一条数据 
  });
}
```
"action"是一个核心概念。他是一个js对象，描述了我们需要做的事和我们需要的数据。如上所述，我们要做的是添加一条数据，我们需要的数据是一个名叫"Marco"的name。

## Your "Store" Responds to Dispatched Actions(store触发回调)
store也是一个核心概念。我们在应用中创建一个集合，存放方法和数据，它通常是一个列表。

store是单例的，在你的整个应用中只有一个store存在：
``` js
// Single object representing list data and logic
var ListStore = {  
// Actual collection of model data 
  items: []
};
```
store会对被分发的action做出处理：
``` js
var ListStore = ...

//在dispatcher中注册我们需要监听的一些事件
AppDispatcher.register( function( payload ) {
    switch( payload.actionName )｛
        //对action作出处理
        case 'new-item':
            //存储一个数据
            ListStore.items.push(payload.newItem);
            break;
   }
});
```
以上是一个典型的例子，介绍了Flux处理回调函数的方式。传入的参数payload包含了action的名称和需要处理的数据。
switch语句中会对相应的action做出数据的处理。

关键概念：
 - store不是MVC中的model，但是它包含了models。
 - 应用中数据处理只能在store中进行，这是Flux核心理念。被分发的action无法增加或者删除一条数据。

假设，你的应用中需要保存一些图片及其基本信息，那么你应该再创建一个Items，命名为ImageItems。一个数组即可代表一种数据类型了。

只有stores能够注册被分发action的回调函数。千万不要在视图中调用*AppDispatcher.register*。dispatcher只会单向地从视图传递数据给store。视图会针对不同的事件重新渲染。

## Your Store Emits a "Change" Event(store触发change事件)
现在，store中的数据已经改变了，我们传递出数据改变的信息了。

我们将让store出发一个事件，如果在你使用了[MicroEvent.js](http://notes.jetienne.com/2011/03/22/microeventjs.html)：
``` js
MicroEvent.mixin( ListStore );
```
然后触发change事件：
``` js
case 'new-item':
     ListStore.items.push( payload.newItem ); 
  
   // Tell the world we changed! 
     ListStore.trigger('change');
     break;
```
关键概念：
 - 事件触发的时候不传递数据，视图只关心是否有数据发生变化。

## Your View Responds to the "Change" Event（视图接收到事件重新渲染）
视图会在数据发生变化后重新渲染，没错是重新渲染。

我们在react组件的初始化完成后为store注册监听：
``` js
componentDidMount: function( ) {
    ListStore.bind( 'change', this.listChanged );
},
```
为了简单起见，我们调用forceUpdate，使视图重新渲染。
``` js
listChanged : function() {
    this.forceUpdate( );
},
```
别忘记在组件回收时，解绑监听的事件:
``` js
componentWillUnmount: function( ){
      ListStore.unbind( 'change', this.listChanged );
},
```
然后来看下组件的render函数:
``` js
render: function() {  
    // Remember, ListStore is global!  
    // There's no need to pass it around 
    var items = ListStore.getAll();  
    // Build list items markup by looping  
    // over the entire list 
    var itemHtml = items.map( function( listItem ) {  
      // "key" is important, should be a unique  
      // identifier for each list item 
        return <li key={listItem.id}>
              {listItem.name}
                    </li>; 
      });
     return <div> 
                  <ul> {itemHtml} </ul> 
                  <button onClick={this.createNewItem}>New Item</button> 
     </div>;
}
```

我们已经完成了整个数据，视图更新过程.当你添加一条数据的时候，视图会分发一个action，store会对action做出数据处理，并且触发一个change事件，之后视图接受到change事件并重新渲染。

但是有一个问题，每当数据更新的时候，视图将全部重新渲染，这样是否会造成效率低下？
其实并不会发生你所担心的事情，React内部构建了虚拟DOM，react会比较视图是否发生变化，实现部分的渲染，是以JS计算开销换取了dom渲染开销的方法来提升效率。

## One More Thing: What The Hell Is An "Action Creator"?（Action Creator是什么鬼？）
我们在点击按钮的时候触发了一个action
``` js
AppDispatcher.dispatch({
     eventName: 'new-item', 
     newItem: {name: 'Samantha'}
 });
```
当我们拥有很多按钮，需要出发不同的action时，我们需要这么写看起来会比较优雅：
  ``` js
ListActions = { 
  add: function( item ) { 
    AppDispatcher.dispatch({ 
        eventName: 'new-item', 
        newItem: item 
    }); 
  }

 del: ...

};
```
现在增加一条数据的时候就只需要这么写：*ListActions.add({name: '...'})*

## PS:不要使用 *forceUpdate*
文中只为了简单起见，使用了*forceUpdate*，正确的做法应该从store中读取数据，并且改变state，触发视图更新。
