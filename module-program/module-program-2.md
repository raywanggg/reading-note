## 模块化编程（二）
### AMD规范
#### 一、why important?
javascript模块规范有两种：[CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1) 和 [AMD](https://github.com/amdjs/amdjs-api/wiki/AMD)。

node.js的 [模块系统](http://nodejs.org/docs/latest/api/modules.html)，就是参照CommonJS规范实现的；
在CommonJS中，有一个全局性方法require()，用于加载模块。

	加载：var math = require('math');

	调用：math.add(2,3);//5
#### 二、浏览器环境
上节代码如果在浏览器中运行会有很大的问题：第二行要在第一行require（'math'）之后运行，因此必须等待math.js加载完成，也就是说如果加载时间很长，应用会停在那里！这对于服务器端不是一个问题，因为所有的模块都放在本地硬盘，可以同步加载完成，等待的时间只是硬盘的读取时间；但是对于浏览器，模块都放在服务器端，等待时间取决于网速的快慢，可能等待时间较长，浏览器处于“假死”状态。

因此，浏览器端的模块，不能采用“同步加载”（synchronous），只能采用“异步加载”（asynchronous）；因此CommonJS不适用于浏览器环境，AMD由此诞生。
#### 三、AMD
AMD是"Asynchronous Module Definition"的缩写，意思就是"异步模块定义"。它采用异步方式加载模块，模块的加载不影响它后面语句的运行。所有依赖这个模块的语句，都定义在一个回调函数中，等到加载完成之后，这个回调函数才会运行。

AMD也采用require（）语句加载模块，不同于CommonJS的是要求两个参数：

	require（[module],callback）;

第一个参数［module］是一个数组，里面的成员就是要加载的模块；
第二个参数callback则是加载成功之后的回调函数；

改成AMD形式如下：
	
	require(['math'],function(math) {
	math.add(2,3);
	});
	
math.add()与math模块加载不是同步的，浏览器不会发生假死。所以很显然，AMD比较适合浏览器环境。
目前，主要有两个Javascript库实现了AMD规范：[require.js](http://requirejs.org) 和 [curl.js](https://github.com/cujojs/curl)。
