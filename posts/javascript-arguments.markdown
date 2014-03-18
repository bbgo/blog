---
layout: post
title: "JavaScript的参数传递"
date: 2013-11-19 23:54:42 +0800
comments: true
categories: javascript
---

### 一、前言

ECMA中所有函数的参数都是按照*值传递*。

### 二、基本数据类型

在向函数传递基本数据类型的时候，被传递的值会被复制给一个局部变量（arguments对象的一个元素）。

	function test( x ) {
		x = 10; //第一次赋值
		console.log( 'x1 : ' + x );	
		console.log( 'arguments1 : ' + arguments )
		console.log( 'arguments[0]1 : ' + arguments[0] )
		
		arguments[0] = 100;
		
		console.log( 'x2 : ' + x );	
		console.log( 'arguments2 : ' + arguments )
		console.log( 'arguments[0]2 : ' + arguments[0] )
	};
	
	test(5);

<!--more-->

    function test( num ) {
        num = num + 10;
        return num;
    };
	
    var num = 20;
    var result = test( num );
    console.log( 'num : ' + num );
    console.log( 'result : ' + result );
	
### 三、引用数据类型

在向函数传递引用数据类型的时候，会把这个值的内存地址复制给一个局部变量。所以局部变量的变化会反映到函数外部。

    function setName( obj ) {
        obj.name = '卢林';
    };
	
    var lou = new Object();
    window.setName( lou );
    console.log( lou.name );

    function setName( obj ) {
        obj.name = '卢林';
        //重新new
        obj = new Object();
        obj.name = 'others'; //其实已经不是传入的内存地址
    };
	
    var lou = new Object();
    window.setName( lou );
    console.log( lou.name );

其实原理很简单，引用类型传入的是内存地址的值（当然JavaScript操作内存地址），比如是8bit，内存地址是：1100 1010，指向的内容是person对象。所以你修改了name属性会影响到函数外部。 