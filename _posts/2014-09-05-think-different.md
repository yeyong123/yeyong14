---
layout: post
title: "Nodejs学习笔记"
description: "Nodejs"
category: "javascript"
tags: ['Nodejs']
---
{% include JB/setup %}
理解Nodejs的回调
======
```js
//假如有一个按钮，点击这个元素隐藏， 在隐藏之后需要告诉用户怎么恢复隐藏的元素。
// 如果按照步骤来写的话就会写成同步的，现在用回调来实现
$('click').hidden('slow',function(){
		alert("The element is hidden;")
		});

//same
$('click').hidden('slow', show());
function show() {
	alert("The element is hidden;")
};
```

### `Javascript`中闭包与匿名函数的理解
------

> 
闭包是值：
有权访问另一个函数作用域的变量对象，创建闭包的常见方式，就是在一个函数内部
创建另一个函数。下面的例子：

```js
	function createFunction(propertype){
		return function(object1, object2){
			var value1 = object1[propertype];
			var value2 = object2[propertype];
			if (value1 < value2) {
				return -1;
			} else if (value1 > value2) {
				return 1;
			} else {
				return 0;
			}
		};
	}
```
>
在上面的代码中`var value1= object1[propertype]` and `var value2 =
object2[propertyoe]`，这两行代码访问了外部的函数中的变量`propertype`,即使这个
内部函数被返回了，而且是在其他地方被调用了，但它仍然可以访问变量`propertype`，
之所以还能够访问那个变量，是因为内部函数的作用域链中包含`createFunction()`的
作用域。

### 关于this对象
----------
```js
var name = "This window";
var object = {
	name: "My Object",
	getNameFunc: function() {
		return function() {
			return this.name;
		};
	}
}
console.log(object.getNameFunc()());
```
>
这里的`this`返回是全局变量`name`的值`This window`;闭包函数访问了全局变量，没有
访问到局部变量中的`name`的值；每个函数在被调用的时候都会自动取得两个特殊的
变量：`this`and `arguments`内部函数在搜索这两个变量的时候，只会搜索其活动
对象为止，当把外部作用域中的`this`对象保存在一个闭包能够访问的变量中，就可以让
闭包访问该对象了。



>
修改上面的代码，让其访问内部的变量：


```js
var name = "This window";
var object = {
	name: "My object",
	getNameFunc: function() {
		var that = this;
		return function() {
			return that.name;
		};
	}
}
console.log(object.getNameFunc()());
```

### 定义私有变量和访问私有变量
------
```js
function MyObject {
	var privateVariable = 10;
	function privateFunction() {
		return false;
	}
	
	//定义函数为闭包，就有权访问在定义在构造函数中德所有变量和函数了。
	this.publicFunction = function() {
		privateVariable++; //定义每次调用这个函数自增一次；
		return privateFunction(); //访问这个私有函数；
	}
}
var myobject = new MyObject();
console.log(myobejct.publicFunction());
			
```

