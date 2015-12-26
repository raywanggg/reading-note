## 模块化编程（一）
### 模块的写法
#### 一、原始写法

function m1() {

//...

}

function m2() {

//...

}

直接调用但污染全局变量，无法保证不与其它模块发生变量名冲突；
#### 二、对象写法

var module1 = new object ({

_count : 0,

m1 : function() {

//...

},

m2 : function() {

//...
 
}
  
});

m1()和 m2()都封装在module1对象里面，使用函数相当于调用这个对象的属性；

例如：module1.m1();

但是外部代码会改变内部计数器的值，例如module._count = 5;
#### 三、立即执行函数写法

var module1 = (function() {

var _count = 0;

var m1 = function() {

//...

};

var m2 = function() {

//...

};(注意var定义类型时候使用；分隔开)

return {

m1 : m1,

m2 : m2

};

})();

module1就是javascript模块的基本写法；此时使用函数外部代码无法读取内部的_count变量；

例如consolo.info(module1._count);//undefined
#### 四、放大模式（augmentation）

前提：一个模块很大必须分成几个部分，或者一个模块需要继承另一个模块，就要采用放大模式：

var module1 = (function (mod) {

mod.m3 = function () {

//...立即执行函数的m1 m2部分

};

return mod;

})(module);

上面代码为module1模块添加一个新方法m3(),然后返回新的module1模块；
#### 五、宽放大模式（Loose augmentation）

前提：有时无法知道哪个部分会先加载，放大模式下第一执行的部分有可能加载一个不存在的空对象，此时采用“宽放大模式”：

var module1 = (function (mod) {

//...

return mod;

})(window.module1 || {});

与放大模式相比，“宽放大模式”就是“立即执行函数”的参数可以是空对象;
#### 六、输入全局变量

独立性是模块化的重要特点，模块内部最好不与程序的其他部分直接交互；为了在模块内部调用全局变量，必须显示地将其它变量输入模块；

var module1 = (function($,YAHOO) {

//...

})(jQuery,YAHOO);

上面的module1模块需要使用jQuery库和YUI库，即相当于把这两个库（其实是两个模块）当作参数输入module1.这样做除了保证模块的独立性，还使得模块之间的依赖关系变得明显。
### summary:模块化写法

**var module1 = (function (mod) {**

**mod.m3 = function () {**	//添加方法m3然后返回新的module1模块

**var _count = 0;**	//弃用对象属性写法采用立即执行函数写法

**var m1 = function () {**

**//...**

**};**

**var m2 = function () {**

**//...**

**};**

**return {**

**m1 : m1,**

**m2 : m2**

**};**

**};**	//m3方法构造完成

**return mod;**
	
**})（window.module1 || {}）;**



