## 模块化编程（三）
### require.js的用法
#### 一、why require.js?
最早时候所有Javascipt代码都写在一个文件里面，加载一个文件就行；后来代码越来越多就分为多个文件依次加载；如下：

	<script src = "1.js"></script>
	<script src = "2.js"></script>
	<script src = "3.js"></script>
	<script src = "4.js"></script>
	...

以上写法的缺点：

1、加载的时候，浏览器会停止网页渲染，加载文件越多，网页失去响应时间越长；

2、js文件之间存在依赖关系，因此必须严格保证加载顺序，且依赖性最大的模块一定要放在最后加载，当依赖关系很复杂的时候，代码编写和维护比较困难；

require.js的诞生就是为了解决这个问题。
#### 二、require.js的加载
1、官网[下载](http://requirejs.org/docs/download.html)最新版本;

2、加载此文件避免网页失去响应的方法一将其放在网页底部加载；二写成如下形式：

	<script src = "js/require.js" defer async = "true"></script>
	
async属性表明这个文件需要异步加载，defer是IE支持的属性。

3、之后加载自己的代码(js文件夹下的main.js文件)形式如下：

	<script src = "js/require.js" data-main = "js/main"></script>
	
data-main属性的作用是指定网页程序的主模块，即main.js会第一个被require.js加载，require.js默认文件后缀是js，所以main.js简写成main。
#### 三、主模块的写法
上一节的main.js，我把它称为"主模块"，意思是整个网页的入口代码。它有点像C语言的main（）函数，所有代码都从这儿开始运行。要用到AMD规范定义的require（）函数的情况是主模块依赖于其他模块。

	//main.js
		require(['moduleA','moduleB','moduleC'],function(moduleA,moduleB,moduleC) {
	
	//some code here

	});
	
require（）函数接受两个参数，第一个是数组，表示所依赖的模块，上例就是['moduleA','moduleB','moduleC'],即主模块依赖这三个模块；第二个参数是一个回调参数，当前面指定的模块都加载成功后，它将被调用。加载的模块会以参数形式传入该函数，从而在回调函数内部就可以使用这些模块。

require()异步加载moduleA，moduleB和moduleC，浏览器不会失去响应；它指定的回调函数，只有前面的模块都加载成功后，才会运行，解决了依赖性的问题；上述代码的实例如下：

	require(['jquery', 'underscore', 'backbone'], function($, _, Backbone) {
	
	//some code here
	
	});
#### 四、模块的加载
上节最后实例中，主模块的依赖模块是['jquery', 'underscore', 'backbone']，默认情况下require.js假定这三个模块与main.js在同一个目录，文件名分别为jquery.js,underscore.js和backbone.js，然后自动加载。

require.config()方法我们可以对模块的加载行为进行自定义；require.config()就写在主模块（main.js）的头部。参数就是一个对象，这个对象的paths属性指定各个模块的加载路径。

	require.config({
		path:{
			"jquery": "lib/jquery.min",
			"underscore": "lib/undercsore.min",
			"backbone": "lib/backbone.min"
		}
	});//此时三个模块默认在js／lib目录之下

另一种情况直接改变基目录（baseUrl）：

	require.config({
		baseUrl: "js/lib",//改变基目录
		path:{
			"jquery": "lib/jquery.min",
			"underscore": "lib/undercsore.min",
			"backbone": "lib/backbone.min"
		}
	});
再一种情况是模块在另一台主机上，可以指定它的网址如下：

	require.config({
		path: {
			"jquery": "https://ajax.googleapis.com/ajax/libs/jquery/1.7.2/jquery.min"
		}
	});	
	
require.js要求每个模块是一个单独的js文件，如果加载多个模块就会发出多个http请求影响网页加载的速度，因此提供了一个[优化工具](http://requirejs.org/docs/optimization.html)，当模块部署完毕之后可以用这个工具将多个模块合并在一个文件中，以减少http的请求。
#### 五、AMD模块的写法
require.js加载的模块采用AMD规范。具体来说就是模块必须采用特定的define（）函数来定义，如果一个模块不依赖其他模块，难么可以直接定义在define（）函数之中。

假定现有一个math.js文件，它定义了一个math模块，则math.js如下：

	//math.js
	define(function() {
		var add = function(x,y) {
			return x+y;
		};
		return {
			add: add
		};
	});
	
加载方法如下：

	//main.js
	require(['math'],function(math) {
		alert(math.add(1,1));
	});

如果这个模块还依赖其他模块，那么define()函数的第一个参数，必须是一个数组，指明该模块的依赖性。

	define(['mylib'], function(myLib) {
		function foo() {
			myLib.doSomething();
		}
		return {
			foo: foo;
		}
	});	

require（）函数加载上面这个模块的时候，就会先加载myLib.js文件。
#### 六、加载非规范的模块
理论上，require.js加载的模块，必须是按照AMD规范、用define()函数定义的模块。实际上，虽然一部分流行的函数库（jQuery）符合AMD规范，更多的库（underscore 和 backbone）并不符合。
非规范的模块在用require（）加载之前，要先用require.config（）方法定义它们的一些特征，例如：

	require.config({
		shim: {
			'underscore': {
				exports: '_'
			},
			'backbone': {
				deps: ['underscore','jquery'],
				exports: 'Backbone'
			}
		}
	});

require.config()接受一个配置对象，这个对象除了前面说过的paths属性之外，还有一个shim属性，专门用来配置不兼容的模块。具体说，每个模块定义1）exports值（输出的变量名），表明这个模块外部调用时的名称；2）deps数组，表明该模块的依赖性。jQuery的插件定义如下：

	shim: {
		'jquery.scroll': {
			deps: ['jquery'],
			exports: 'jQuery.fn.scroll'
		}
	}
#### 七、require.js插件
require.js[插件](https://github.com/jrburke/requirejs/wiki/Plugins) 可以实现一些特定的功能。
domready插件可让回调函数在页面DOM结构加载完成后再运行；

	require(['domready!'],function(doc) {
		//called once the DOM is ready
	});
	
text 和 image插件则是允许require.js加载文本和图片文件；

	define([
		'text!review.txt',
		'image!cat.jpg'
		],
		function(review,cat) {
			consolo.log(review);
			document.body.appendChild(cat);
		}
	);
	
类似的插件还有json和mdown，用于加载json文件和markdown文件;

#### Summary
	
	1、加载行为的定义（require.config()放在main.js的头部）:
	
	require.config({
		baseUrl: "js/lib",//改变基目录
		path:{
			"jquery": "lib/jquery.min",
			"underscore": "lib/undercsore.min",
			"backbone": "lib/backbone.min"
		}
	});
	
	2、主模块（main.js的主体）:
	
	require(['jquery', 'underscore', 'backbone'],
	function($, _, Backbone) {
		//some code here
	});
	
	3、模块的AMD规范 (例如上述加载的三个js文件对应三个要加载的模块):
	3.1、模块的定义（math.js）
	
	define(['mylib'], function(myLib) {//第一个参数为数组指明依赖性
		function foo() {
			myLib.doSomething();
		}
		return {
			foo: foo;
		}
	});	
	
	3.2、模块在主模块中的加载（main.js）

	require(['math'],function(math) {
		alert(math.add(1,1));
	});