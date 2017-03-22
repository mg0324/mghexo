---
title: '新技术重点-commonjs,es6,react'
date: 2017-02-20 16:45:53
categories: react
tags:
- react
- commonjs
- es6
---

## commonjs
百度百科：
>CommonJS API定义很多普通应用程序（主要指非浏览器的应用）使用的API，从而填补了这个空白。它的终极目标是提供一个类似Python，Ruby和Java标 准库。

自己的理解：
>有点像Java的jdk，一种javascript的工具集。

### module.exports
>CommonJS规范规定，每个模块内部，module变量代表当前模块。这个变量是一个对象，它的exports属性（即module.exports）是对外的接口。加载某个模块，其实是加载该模块的module.exports属性。

	var x = 5;
	var addX = function (value) {
	  return value + x;
	};
	module.exports.x = x;
	module.exports.addX = addX;

>上面代码通过module.exports输出变量x和函数addX。

### require
>require方法用于加载模块。

	var example = require('./example.js');

	console.log(example.x); // 5
	console.log(example.addX(1)); // 6



## es6
>ECMAScript 6.0（以下简称 ES6）是 JavaScript 语言的下一代标准，已经在2015年6月正式发布了。它的目标，是使得 JavaScript 语言可以用来编写复杂的大型应用程序，成为企业级开发语言。

### class(extends)类，继承
	
	class Cat { 
	  constructor(name) {
	    this.name = name;
	  }
	  
	  speak() {
	    console.log(this.name + ' makes a noise.');
	  }
	}

	class Lion extends Cat {
	  speak() {
	    super.speak();
	    console.log(this.name + ' roars.');
	  }
	}

>子类继承父类后，会拥有父类的属性和方法。
### ...props
>
### function简写

	()=>{

	}


## react
>React是Facebook开发的一款JS库，实现了es6的语法。

>和其他一些js框架相比，React怎样，比如Backbone、Angular等。

* React不是一个MVC框架，它是构建易于可重复调用的web组件，侧重于UI, 也就是view层
* 其次React是单向的从数据到视图的渲染，非双向数据绑定
* 不直接操作DOM对象，而是通过虚拟DOM通过diff算法以最小的步骤作用到真实的DOM上。
* 不便于直接操作DOM，大多数时间只是对 virtual DOM 进行编程

### 变量(state,props)
>state是只读数据

>props是可变数据

>可以用 `this.setState({dataSource:props.dataSource});` 来变向改变，会刷新render方法，
重新渲染一次。

### 生命周期
>详细请参照 http://react-china.org/t/react/1740


常用的就下面几个：

* componentDidMount
>真实的DOM被渲染出来后调用，在该方法中可通过this.getDOMNode()访问到真实的DOM元素。此时已可以使用其他类库来操作这个DOM。
在服务端中，该方法不会被调用。

* render
>必选的方法，创建虚拟DOM，该方法具有特殊的规则：
	* 只能通过this.props和this.state访问数据
	* 可以返回null、false或任何React组件
	* 只能出现一个顶级组件（不能返回数组）
	* 不能改变组件的状态
	* 不能修改DOM的输出

* componentWillReceiveProps
>组件接收到新的props时调用，并将其作为参数nextProps使用，此时可以更改组件props及state。


    componentWillReceiveProps: function(nextProps) {
        if (nextProps.bool) {
            this.setState({
                bool: true
            });
        }
    }

* componentWillUnmount
>组件被移除之前被调用，可以用于做一些清理工作，在componentDidMount方法中添加的所有任务都需要在该方法中撤销，比如创建的定时器或添加的事件监听器。

