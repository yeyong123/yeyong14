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
