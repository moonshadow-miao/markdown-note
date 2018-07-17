##一. 解释下 JavaScript 中 this 是如何工作的?

- 作为函数调用，this 绑定全局对象，浏览器环境全局对象为 window 。
- 作为方法调用,this指向方法的调用者。
- 作为构造函数使用，this 绑定到新创建的对象。
- 作为对象方法使用，this 绑定到该对象。
- 使用apply或call调用 this 将会被显式设置为函数调用的第一个参数。

##二. js的数据类型和特点  

- 基本类型:Undefined、Null、Boolean、Number、String、es6新增:Symbol(创建后独一无二且不可变的数据类型 )  
- 基本类型特点: 基本类型的值在内存中是按值访问的,存放在栈内存中,两个基本类型的变量比较就是值的比较   
- 引用类型: object   
- 常见本地对象: Array,Function,Date,RegExp,Error,Number,String,Boolean,es6新增:Set,Map,Promise,WeakMap,WeakSet,Generatror,Proxy,Reflect
- 引用类型的特点: 引用类型的值是按引用访问的, 存放在堆内存中 , 两个基本类型的变量比较就是引用的比较(意思就是两个引用是否同时指向同一个对象)

##三. js的原型  
JavaScript 对象有一个指向一个原型对象的链。当试图访问一个对象的属性时，它不仅仅在该对象上搜寻，还会搜寻该对象的原型，以及该对象的原型的原型，依次层层向上搜索，直到找到一个名字匹配的属性或到达原型链的末尾。

有个o对象,var o = {x:1}, o的原型 , o.prototype = {x:2,y:1,getX = function(){return x}} , 问,o.x = ? o.y = ? for ... in 循环o对象,会遍历到哪些属性 ? 如何判断对象上的属性是私有属性还是访问原型获取 ? 

答案: o.x = 1 ,o.y = 1 , for...in循环o对象会遍历o对象上可枚举的属性(自有属性,原型上的属性,这里有x,y,getX),obj.hasOwnProperty判断属性是否是对象的自有属性,不会在原型链上查找 . Object.getOwnPropertyNames获取对象上自有属性(可枚举的),不会获取原型上的属性.

##四: new 操作符 ,new Foo()在创建对象的时候做了什么?

1. 一个继承自 Foo.prototype 的新对象被创建
2. 将 this 绑定到新创建的对象
3. 执行构造函数Foo里面的代码
4. 返回创建的对象(前提是构造函数没有显式的返回其他对象, 意思就是Foo函数里面没有写return)

##五: 闭包?
当函数可以记住并访问所在的词法作用域时， 就产生了闭包， 即让函数是在当前词法作用域之外执行.   
简单的理解就是函数保存着外部作用域的变量的引用(正常情况,函数执行结束,销毁所有的局部变量,如果内层函数还保持着对外层函数的局部变量的引用,这个局部变量就不会被销毁,形成闭包)  
经典的例子,一个for循环,循环变量是i,循环四次,里面是一个定时器,定时器一秒后打印i, 问打印结果? 

	for(var i = 0; i< 3;i++){
		setTimeout(function(){
			console.log(i)
		},1000)
	}

答案: 同时打印了4次4 ,因为定时器延迟执行, 开启的4个定时器打印的是同一个变量i,这里的i是全局变量, i循环结束后值为4 , 所以延迟执行的4个定时器打印的都是4 .
追问: 如何实现打印0,1,2,3 ?
思路:每次循环都要创建一个作用域
方法: 

	for(var i = 0; i< 3;i++){
		(function(index){
			setTimeout(function(){
				console.log(index)
			},1000)
		})(i)
	}

简单实现 , 使用es6的let声明i , let的原理和上面的代码一样( let让每次for循环都会生成一个独立作用域 )

##六 : DOM 操作,添加,移除,替换,复制,创建,查找, js原生方法,jq方法?

1. 创建: 原生:createDocumentFragment()    //创建一个DOM片段  
    createElement()   //创建一个具体的元素  
    createTextNode()   //创建一个文本节点  
	jq:  $("<div></div>")
2. 添加 : 原生: appendChild()  , insertBefore()
	jq: append() , prepend(), before() ,after(),insertAfter(),insertBefore()  
3. 移除: 原生: removeChild()
	jq: reomve() empty()
4. 替换: 原生: replaceChild()
	jq: replaceWith(), replaceAll()
5. 复制: 原生: cloneNode() jq:clone()
6. 查找: 原生: getElementsByTagName()  getElementsByName() getElementById() getElementsByClass() querySelector() querySelectorAll() 
	jq: $()
##七: jq的on方法有哪些用法?如何实现事件委托(原理)?

  1.绑定事件 on('click',function(){})  
  2.绑定多个事件不同函数

	on({
	 	mouseover:function(){$("body").css("background-color","lightgray");}, 
		mouseout:function(){$("body").css("background-color","lightblue");}
	})
  3 多个事件绑定同一个函数: on('click mouseover',function(){})  
  4 绑定自定义事件: on('myEvent',function(){}) 
  5 传递数据到函数:
	
	 on("click", {msg: "hello"}, function(e){
		console.log(e.data.msg)  // hello
	}) 
  6 给动态创建的元素绑定事件(事件委托,原理事件冒泡,给父元素绑定事件,由子元素触发冒泡到父元素) 　$("div").on("click","p",function(){})

##八 事件流有哪些?事件捕获和事件冒泡的过程   
事件流：从页面中接收事件的顺序。也就是说当一个事件产生时，这个事件的传播过程，就是事件流 .处理事件流有两种,事件捕获和事件冒泡 . “DOM2级事件”规定事件包括三个阶段：事件捕获阶段==>处于目标阶段==>事件冒泡阶段
  
事件捕获：Netscape团队提出的另一种事件流叫事件捕获，事件捕获的思想是不太具体的节点应该更早接收到事件，而最具体的节点应该最后接收到事件。 

事件冒泡:IE的事件流叫事件冒泡，即事件开始时由最具体的元素(文档中嵌套层次最深的节点)接收，然后逐级向上传播到较为不具体的节点。

'DOM2级事件'定义了两个方法，用于处理指定和删除事件处理程序的操作：addEventListener()和removeEventListener() ,接收3个参数：要处理的事件名、作为事件处理程序的函数,问第三个参数是什么?默认是什么值?

答案: 第三个参数是布尔值,控制事件处理程序,默认是false,事件冒泡. 如果设置为true,则是事件捕获.

##九: http协议相关 
url的组成部分?分析http://www.baidu.com:8080/news?time=174411#name的组成?

1. 协议http 
1. 域名 www.baidu.com (顶级域名.com,主域名baidu.com,二级域名,www.baidu.com)  
1. 端口号  
1. 路径/news
1. 查询参数 ?time=1777454 
1. 锚点 #name

说几个常见的状态码? 

- 200：请求成功  
- 201：请求成功并且服务器创建了新的资源
- 302：服务器目前从不同位置的网页响应请求，但请求者应继续使用原有位置来响应以后的请求。
- 304：自从上次请求后，请求的网页未修改过。服务器返回此响应时，不会返回网页内容。
- 400：服务器不理解请求的语法。
- 404：请求的资源（网页等）不存在
- 500： 内部服务器错误 

http请求包括哪几部分?说几个常见的请求头,响应头字段和值?  
请求行（request line）、请求头部（header）、空行(不做要求)和请求数据四个部分组成  

- Content-Type: 指定字符编码,值 text/plain,text/html application/x-www-form-urlencoded(post请求)
- Cookie 
- Cache-Control 对缓存进行控制, 值: no-cache 
- Content-Encoding 响应内容的压缩方式, 值 gzip deflate
- Connection 连接方式 , 值keep-alive ,close
- Modified-Since 服务端判断此值未变或不需要更新时返回304，表明客户端可直接使用缓存. 值,日期格式

##十: css相关  
css盒模型?   
标准盒模型：宽度=内容的宽度（content）+ border + padding + margin
低版本IE盒模型：宽度=内容宽度（content+border+padding）+ margin

css盒模型对应css3哪个属性?值分别是什么?
box-sizing属性 ,context-box：W3C的标准盒子模型，border-box：IE传统盒子模型。

如何使用css创建一个三角形?   
思路: 利用border,隐藏三个边的border

	width: 0;
	height: 0;
	border-top: 10px solid transparent;
	border-left: 10px solid transparent;
	border-right: 10px solid transparent;
	border-bottom: 10px solid #ff0000;

CSS选择符有哪些？哪些属性可以继承？
 *   1.id选择器（ # myid）
  	2.类选择器（.myclassname）
  	3.标签选择器（div, h1, p）
  	4.相邻选择器（h1 + p）
  	5.子选择器（ul > li）
  	6.后代选择器（li a）
  	7.通配符选择器（ * ）
  	8.属性选择器（a[rel = "external"]）
  	9.伪类选择器（a:hover, li:nth-child）

  *  常见可继承的样式： color font-xxxx(font-size,font-weight```) text-xxx(text-align,text-indent)  line-height visibility
## 事件队列
	setTimeout(() => console.log('a'), 0);
	var p = new Promise((resolve) => {
	  console.log('b');
	  resolve();
	});
	p.then(() => console.log('c'));
	p.then(() => console.log('d'));
	console.log('e');
	// 结果：b e c d a
	// 任务队列优先级：promise.Trick()>promise的回调>setTimeout>setImmediate
	async function async1() {
	    console.log("a");
	    await  async2(); //执行这一句后，await会让出当前线程，将后面的代码加到任务队列中，然后继续执行函数后面的同步代码
	    console.log("b");  // 相当于 Promise.resolve(console.log('b'))
	
	}
	async function async2() {
	   console.log( 'c');
	}
	console.log("d");
	setTimeout(function () {
	    console.log("e");
	},0);
	async1();
	new Promise(function (resolve) {
	    console.log("f");
	    resolve();
	}).then(function () {
	    console.log("g");
	});
	console.log('h');
## 手写bind 
	Function.prototype.bind2 = function (context) {

    if (typeof this !== "function") {
      throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
    }

    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);
    var fNOP = function () {};

    var fbound = function () {
        self.apply(this instanceof self ? this : context, args.concat(Array.prototype.slice.call(arguments)));
    }

    fNOP.prototype = this.prototype;
    fbound.prototype = new fNOP();

    return fbound;

}
## 浏览器的重绘和重排

![](https://pic1.zhimg.com/80/e8bc40d7006f13fa0a191d774b7db36a_hd.jpg)  

重绘:当 DOM 元素的属性发生变化 (如 color) 时, 浏览器会通知 render 重新描绘相应的元素, 此过程称为 repaint。  
重排:如果该次变化涉及元素布局 (如 width), 浏览器则抛弃原有属性, 重新计算并把结果传递给 render 以重新描绘页面元素, 此过程称为 reflow（回流）

常见的触发重排的操作:  

-  页面初始渲染
-  添加/删除可见DOM元素
-  改变元素位置
-  改变元素尺寸（宽、高、内外边距、边框等）
-  改变元素内容（文本或图片等）
-  改变窗口尺寸
-  添加或者删除可见的DOM元素
-  激活CSS伪类（例如：:hover）
-  查询某些属性或调用某些方法

触发重排的方法:  

- clientWidth、clientHeight、clientTop、clientLeft
- offsetWidth、offsetHeight、offsetTop、offsetLeft
- scrollWidth、scrollHeight、scrollTop、scrollLeft 
- scrollIntoView() 、 scrollIntoViewIfNeeded()
- getComputedStyle()、 getBoundingClientRect() 、 scrollTo()


优化:  

- js 优化：
-  将多次改变样式属性的操作合并成一次操作。
-  将需要多次重排的元素，position属性设为absolute或fixed，这样此元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。
-  在内存中多次操作节点，完成后再添加到文档中去。例如要异步获取表格数据，渲染到页面。可以先取得数据后在内存中构建整个表格的html片段，再一次性添加到文档中去，而不是循环添加每一行。
- 由于display属性为none的元素不在渲染树中，对隐藏的元素操作不会引发其他元素的重排。如果要对一个元素进行复杂的操作时，可以先隐藏它，操作完成后再显示。这样只在隐藏和显示时触发2次重排。 
- 在需要经常取那些引起浏览器重排的属性值时，要缓存到变量。 
- css：优化
- 避免使用table布局。
- 尽可能在DOM树的最末端改变class。
- 避免设置多层内联样式。
- 将动画效果应用到position属性为absolute或fixed的元素上。
- 避免使用CSS表达式（例如：calc()）。
##get和post
常见用法区别: 

- GET 请求可被缓存
- GET 请求保留在浏览器历史记录中
- GET 请求可被收藏为书签
- GET 请求不应在处理敏感数据时使用
- GET 请求有长度限制
- GET 请求只应当用于取回数据
- GET产生一个TCP数据包;POST产生两个TCP数据包。(除了火狐浏览器)
## 如何让  a == 1 && a== 2 && a==3
	var a = {num:0};
	a.valueOf = function(){   // 同时可以用 toString()
	    return ++a.num
	}


## this题目
	var names = "a";
	var obj = {
	    names:"b",
	    showName:function(){
	        console.log(this.names);
	    },    
	}
	var newObj = obj.showName.bind(window);
	newObj.call(obj)   //  a  因为bind指向this对象后 再一次调用的话 this指向不会被改变

	function a(a,b,c){
      console.log(this.length);   // 4  实参个数
      console.log(this.callee.length);  // 1  形参个数
    }

    function fn(d){
      arguments[0](10,20,30,40,50);    // argument.0 = a ,所以a 中的this指向argument
    }

    fn(a,10,20,30);

	var x = 1;
	if (function f(){}) {    // if的 ()里面不存在函数提示,只会计算表达式
	x += typeof f;  
	}
	console.log(x); 
## RESTFUL API
接口拥有安全性和幂等性的特性，例如GET和HEAD请求都是安全的， 无论请求多少次，都不会改变服务器状态
GET、HEAD、PUT和DELETE请求都是幂等的，无论对资源操作多少次， 结果总是一样的，后面的请求并不会产生比第一次更多的影响。   
安全方法是指不修改资源的 HTTP 方法  
HTTP 幂等方法是指无论调用多少次都不会有不同结果的 HTTP 方法。

GET 请求:

- **安全且幂等**
- 获取表示
- 变更时获取表示（缓存）
- 200（OK） 表示已在响应中发出
- 204（无内容） 资源有空表示
- 301（Moved Permanently） 资源的URI已被更新
- 303（See Other） 其他（如，负载均衡）
- 304（not modified）资源未更改（缓存）
- 400 （bad request）指代坏请求（如，参数错误）
- 404 （not found）资源不存在
- 406 （not acceptable）服务端不支持所需表示
- 500 （internal server error）通用错误响应
- 503 （Service Unavailable）服务端当前无法处理请求

POST

-** 不安全且不幂等**
- 使用服务端管理的（自动产生）的实例号创建资源
- 创建子资源
- 部分更新资源
- 如果没有被修改，则不过更新资源（乐观锁）
- 200（OK）如果现有资源已被更改
- 201（created）如果新资源被创建
- 202（accepted）已接受处理请求但尚未完成（异步处理）
- 301（Moved Permanently）资源的URI被更新
- 303（See Other）其他（如，负载均衡）
- 400（bad request）指代坏请求
- 404 （not found）资源不存在
- 406 （not acceptable）服务端不支持所需表示
- 409 （conflict）通用冲突
- 412 （Precondition Failed）前置条件失败（如执行条件更新时的冲突）
- 415 （unsupported media type）接受到的表示不受支持
- 500 （internal server error）通用错误响应
- 503 （Service Unavailable）服务当前无法处理请求

PUT
- 
- **不安全但幂等**
- 用客户端管理的实例号创建一个资源
- 通过替换的方式更新资源
- 如果未被修改，则更新资源（乐观锁）
- 200 （OK）如果已存在资源被更改
- 201 （created）如果新资源被创建
- 301（Moved Permanently）资源的URI已更改
- 303 （See Other）其他（如，负载均衡）
- 400 （bad request）指代坏请求
- 404 （not found）资源不存在
- 406 （not acceptable）服务端不支持所需表示
- 409 （conflict）通用冲突
- 412 （Precondition Failed）前置条件失败（如执行条件更新时的冲突）
- 415 （unsupported media type）接受到的表示不受支持
- 500 （internal server error）通用错误响应
- 503 （Service Unavailable）服务当前无法处理请求

DELETE

-** 不安全但幂等**
- 删除资源
- 200 （OK）资源已被删除
- 301 （Moved Permanently）资源的URI已更改
- 303 （See Other）其他，如负载均衡
- 400 （bad request）指代坏请求
- 404 （not found）资源不存在
- 409 （conflict）通用冲突
- 500 （internal server error）通用错误响应
- 503 （Service Unavailable）服务端当前无法处理请求
## 从浏览器地址栏输入url到显示页面的步骤(以HTTP为例)
1. 在浏览器地址栏输入URL
2. 浏览器查看缓存，如果请求资源在缓存中并且新鲜，跳转到转码步骤   
   (1)如果资源未缓存，发起新请求  
   (2)如果已缓存，检验是否足够新鲜，足够新鲜直接提供给客户端，否则与服务器进行验证。  
   (3)检验新鲜通常有两个HTTP头进行控制Expires和Cache-Control：  
   (4)HTTP1.0提供Expires，值为一个绝对时间表示缓存新鲜日期   
   (5)HTTP1.1增加了Cache-Control: max-age=,值为以秒为单位的最大新鲜时间    
3. 浏览器解析URL获取协议，主机，端口，path
4. 浏览器组装一个HTTP（GET）请求报文
5. 浏览器获取主机ip地址，过程如下：  
   (1)浏览器缓存  
   (2)本机缓存  
   (3)hosts文件  
   (4)路由器缓存   
   (5)ISP DNS缓存   
   (6)DNS递归查询（可能存在负载均衡导致每次IP不一样）
6. 打开一个socket与目标IP地址，端口建立TCP链接，三次握手如下： 
   (1)客户端发送一个TCP的SYN=1，Seq=X的包到服务器端口  
   (2)服务器发回SYN=1， ACK=X+1， Seq=Y的响应包  
   (3)客户端发送ACK=Y+1， Seq=Z
7. TCP链接建立后发送HTTP请求
8. 服务器接受请求并解析，将请求转发到服务程序，如虚拟主机使用HTTP Host头部判断请求的服务程序
9. 服务器检查HTTP请求头是否包含缓存验证信息如果验证缓存新鲜，返回304等对应状态码
10. 处理程序读取完整请求并准备HTTP响应，可能需要查询数据库等操作
11. 服务器将响应报文通过TCP连接发送回浏览器
12. 浏览器接收HTTP响应，然后根据情况选择关闭TCP连接或者保留重用，关闭TCP连接的四次握手如下：  
   (1)主动方发送Fin=1， Ack=Z， Seq= X报文  
   (2)被动方发送ACK=X+1， Seq=Z报文   
   (3)被动方发送Fin=1， ACK=X， Seq=Y报文   
   (4)主动方发送ACK=Y， Seq=X报文  
13. 浏览器检查响应状态吗：是否为1XX，3XX， 4XX， 5XX，这些情况处理与2XX不同
14. 如果资源可缓存，进行缓存
15. 对响应进行解码（例如gzip压缩）
16. 根据资源类型决定如何处理（假设资源为HTML文档）
17. 解析HTML文档，构件DOM树，下载资源，构造CSSOM树，执行js脚本，这些操作没有严格的先后顺序，以下分别解释
18. 构建DOM树：   
   (1)Tokenizing：根据HTML规范将字符流解析为标记  
   (2)Lexing：词法分析将标记转换为对象并定义属性和规则  
   (3)DOM construction：根据HTML标记关系将对象组成DOM树  
19. 解析过程中遇到图片、样式表、js文件，启动下载
20. 构建CSSOM树：   
   (1)Tokenizing：字符流转换为标记流  
   (2)Node：根据标记创建节点  
   (3)CSSOM：节点创建CSSOM树
21. 根据DOM树和CSSOM树构建渲染树:   
   (1)从DOM树的根节点遍历所有可见节点，不可见节点包括：1）script,meta这样本身不可见的标签。   2)被css隐藏的节点，如display: none   
   (2)对每一个可见节点，找到恰当的CSSOM规则并应用   
   (3)发布可视节点的内容和计算样式
22. js解析如下：  
   (1)浏览器创建Document对象并解析HTML，将解析到的元素和文本节点添加到文档中，此时document.readystate为loading   
   (2)HTML解析器遇到没有async和defer的script时，将他们添加到文档中，然后执行行内或外部脚本。这些脚本会同步执行，并且在脚本下载和执行时解析器会暂停。这样就可以用document.write()把文本插入到输入流中。同步脚本经常简单定义函数和注册事件处理程序，他们可以遍历和操作script和他们之前的文档内容    
   (3)当解析器遇到设置了async属性的script时，开始下载脚本并继续解析文档。脚本会在它下载完成后尽快执行，但是解析器不会停下来等它下载。异步脚本禁止使用document.write()，它们可以访问自己script和之前的文档元素   
   (4)当文档完成解析，document.readState变成interactive   
   (5)所有defer脚本会按照在文档出现的顺序执行，延迟脚本能访问完整文档树，禁止使用document.write()   
   (6)浏览器在Document对象上触发DOMContentLoaded事件   
   (7)此时文档完全解析完成，浏览器可能还在等待如图片等内容加载，等这些内容完成载入并且所有异步脚本完成载入和执行，document.readState变为complete,window触发load事件
23. 显示页面（HTML解析过程中会逐步显示页面）
##跨域
1.通过jsonp跨域
	
2.document.domain + iframe + window.parent跨域   
此方案仅限主域相同，子域不同的跨域应用场景  
两个页面都通过js强制设置document.domain为基础主域，就实现了同域,然后通过window.parent来访问父窗口的内容,实现跨域信息交流

3.window.name + iframe跨域
##如何进行网站性能优化
###content方面
1. 减少HTTP请求：合并文件、CSS精灵、inline Image
1. 减少DNS查询：DNS查询完成之前浏览器不能从这个主机下载任何任何文件。方法：DNS缓存、将资源分布到恰当数量的主机名，平衡并行下载和DNS查询
1. 避免重定向：多余的中间访问
1. 使Ajax可缓存
1. 非必须组件延迟加载
1. 未来所需组件预加载
1. 减少DOM元素数量
1. 将资源放到不同的域下：浏览器同时从一个域下载资源的数目有限，增加域可以提高并行下载量
1. 减少iframe数量
1. 不要404
###Server方面
1. 使用CDN
1. 添加Expires或者Cache-Control响应头
1. 对组件使用Gzip压缩
1. 配置ETag
1. Flush Buffer Early
1. Ajax使用GET进行请求
1. 避免空src的img标签
###Cookie方面
1. 减小cookie大小
1. 引入资源的域名不要包含cookie
###css方面
1. 将样式表放到页面顶部
1. 不使用CSS表达式
1. 使用不使用@import
1. 不使用IE的Filter
###Javascript方面
1. 将脚本放到页面底部
1. 将javascript和css从外部引入
1. 压缩javascript和css
1. 删除不需要的脚本
1. 减少DOM访问
1. 合理设计事件监听器
###图片方面
1. 优化图片：根据实际颜色需要选择色深、压缩
1. 优化css精灵
1. 不要在HTML中拉伸图片
1. 保证favicon.ico小并且可缓存