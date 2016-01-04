## transition
### 使用场合
1、css 伪元素

	.element {
  		height: 100px;
  		transition: height 2s linear;
	}
	//定义属性名称，过渡时长和变化函数
	
	.element:hover {
 		 height: 200px;
	}

没有说明的情况下，浏览器默认property是all；time－function是ease；duration需要定义；

2、不限于伪类，程序性添加和删除class

	/* CSS */
	.element {
  		opacity: 0.0;
  		transform: scale(0.95) translate3d(0,100%,0);
  		transition: transform 400ms ease, opacity 400ms ease;
	}

	.element.active {
  		opacity: 1.0;
  		transform: scale(1.0) translate3d(0,0,0);
	}

	.element.inactive {
  		opacity: 0.5;
  		transform: scale(1.0) translate3d(0,0,0);
	}

	// JS with jQuery
	var active = function(){
  		$('.element').removeClass('inactive').addClass('active');
	};

	var inactive = function(){
  		$('.element').removeClass('active').addClass('inactive');
	};
### 转变梯度
1、[transition support properties](http://oli.jp/2010/css-animatable-properties/) are provided.

2、transition not support background gradients.实现方法是给梯度加上一个透明度，然后再在background color中转变：

	.panel {
  		background-color: #000;
  		background-image: linear-gradient(rgba(255, 255, 0, 0.4), #FAFAFA);
  		transition: background-color 400ms ease;
	}

	.panel:hover {
  		background-color: #DDD;
	}
注意linear-gradient（a,b）默认从上到下的a－b转变；从下到上可以写为linear-gradient（to top,a,b）
### 硬件加速
一些属性例如left，margin的变换会导致浏览器重新计算每一帧的样式，这是非常昂贵的；（优化例子见下节）
### clipping裁剪
1、用translate3d代替宽度

	.clipped {
  		overflow: hidden;
  		position: relative;
	}

	.clipped .clip {
  		right: 0px;
  		width: 45px;
  		height: 45px;
  		background: url(/images/clip.png) no-repeat
	}
	input {
  		-webkit-transition: -webkit-transform 400ms ease;
  	}//样式省略
	input:focus {
  		-webkit-transform: translate3d(-50px, 0, 0);
	}
input ![icon](input.png =120x60)
input:focus ![icon](input-focus.png =150x60)
### timing function时间函数
1、pre-defined timing function: **linear**, **ease**, **ease-in**, **ease-out** and **ease-in-out**.

2、通过指定四点沿立方贝塞尔曲线写我们自己的计时功能；不用猜测数值，而是通过需要的效果的来确定我们需要的数值；一是根据效果确定曲线的 [pre-defined curves](http://easings.net/zh-cn)； 二是[graphing tool](http://cubic-bezier.com/#.17,.67,.83,.67),这个可以根据拖动的球体来确定参数
### programmatic transitions
1、通过js不仅能触发过渡，也能定义它们。

	var defaults = {
  		duration: 400,
  		easing: ''
	};

	$.fn.transition = function (properties, options) {
  		options = $.extend({}, defaults, options);
  		properties['webkitTransition'] = 'all ' + options.duration + 'ms ' + options.easing;
  		$(this).css(properties);
	};
	
	//以上定义transition函数，实例中使用如下：
	
	setTimeout(function(){
  		var callback = function(){ alert('Transition complete') };
  		$('.element').transition({marginLeft: 100}, {complete: callback});
	}, 2000);

还可以像$.fn.animate()那样在参数中使用duration、easing 和 complete如下：
	
	$("#box").transition({
  		opacity: 0.1, scale: 0.3,
  		duration: 500,
  		easing: 'in',
  		complete: function() { /* ... */ }
	});
### transition callback
1、为了确定调用了回调函数，设置一个触发事件的timeout;

	var defaults = {
  		//同上
	};

	$.fn.transition = function (properties, options) {
  		options = $.extend({}, defaults, options);
  		properties['webkitTransition'] = 'all ' + options.duration + 'ms ' + options.easing;
  		$(this).one('webkitTransitionEnd', options.complete || $.noop);
  		$(this).emulateTransitionEnd();
  		$(this).css(properties);
	};
在👆programmatic transitions定义的基础上加上：

	$.fn.emulateTransitionEnd = function(duration) {
  		var called = false, $el = this;
  		$(this).one('webkitTransitionEnd', function() { called = true; });
  		var callback = function() { if (!called) $($el).trigger('webkitTransitionEnd'); }; 		
  		setTimeout(callback, duration);
	};
	
	//实例如下：
	
	setTimeout(function(){
  		var callback = function(){ alert('Transition complete') };
  		$('.element').transition({marginLeft: 100}, {complete: callback});
	}, 2000);
### chaining transitions
1、先要将css过渡放在$.fn.queue 的调用函数之中，然后确保我们触发了$.fn.dequeue；

	var defaults = {
  		//同上
	};
	
	$.fn.transition = function (properties, options) {
  		var $el = $(this);
  		options = $.extend({}, defaults, options);
  		properties['webkitTransition'] = 'all ' + options.duration + 'ms ' + options.easing;
  
  		var callback = function(){
    		$el.dequeue();
    		if (options.complete) options.complete.apply($el);
  		};
    
  		$el.queue(function(){
    		$el.one('webkitTransitionEnd', callback);
    		$el.emulateTransitionEnd();
    		$el.css(properties);
  		});
      
  		return this;
	};
在👆programmatic transitions定义的基础上加上：

	$.fn.emulateTransitionEnd = function(duration) {
  		//同上
	};
	
	//实例如下：
	
	setTimeout(function(){
  		var callback = function(){ alert('Transition complete') };
  		$('.element')
      		.transition({marginLeft: 100})
      		.delay(1000)
      		.transition({marginTop: 200}, {complete: callback});
	}, 2000);
### redrawing
1、过渡有两种状态的css属性，一个是初始状态动画应该有的，另一个是动画终止时候的属性；但如果像下面这样表示：

	$('.element').css({left: '10px'}).transition({left: '20px'});
一个执行完会立即执行另外一个，浏览器会加速渲染的速度优化属性变化的过程以至于没有变化；解决方法就是使用DOM element’s offsetHeight property

	$.fn.redraw = function(){
  		$(this).each(function(){
    		var redraw = this.offsetHeight;
  		});
	};
	//应用如下
	$('.element').css({left: '10px'})
             .redraw()
             .transition({left: '20px'});