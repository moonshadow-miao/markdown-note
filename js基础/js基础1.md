#在HTML中使用javascript
###script标签的defer和async
1. 没有 defer 或 async，浏览器会立即加载并执行指定的脚本，“立即”指的是在渲染该 script 标签之下的文档元素之前，也就是说不等待后续载入的文档元素，读到就加载并执行。
2. async,加载和渲染后续文档元素的过程将和 script.js 的加载与执行并行进行（异步）。
3. defer,加载后续文档元素的过程将和 script.js 的加载并行进行（异步），但是 script.js 的执行要在所有元素解析完成之后，DOMContentLoaded 事件触发之前完成。
![async,defer](http://segmentfault.com/img/bVcQV0)
*蓝色线代表网络读取，红色线代表执行时间，这俩都是针对脚本的；绿色线代表 HTML 解析* 
 
###比较:

- efer 和 async 在网络读取（下载）这块儿是一样的，都是异步的
- 它俩的差别在于脚本下载完之后何时执行，显然 defer 是最接近我们对于应用脚本加载和执行的要求的
- 关于 defer，它是按照加载顺序执行脚本的,async则是乱序执行,不管声明的顺序如何，只要加载完了就会立刻执行

书签:p38

###undefined与null的区别:  

- 在 if 中，undefined 和 null 都自动转译为 false。
- 在转换为数字的过程中，undefined 转译为 NaN， null 转译为0。
- null 表示"没有对象"，即该处不应该有值。
- undefined 表示"缺少值"，就是此处应该有一个值，但是还没有定义。

### get和post 请求的区别

- GET在浏览器回退时是无害的，而POST会再次提交请求。
- GET产生的URL地址可以被Bookmark，而POST不可以。 
- GET请求会被浏览器主动cache，而POST不会，除非手动设置。
- GET请求只能进行url编码，而POST支持多种编码方式。
- GET请求参数会被完整保留在浏览器历史记录里，而POST中的参数不会被保留。 
- GET请求在URL中传送的参数是有长度限制的，而POST么有。 
- 对参数的数据类型，GET只接受ASCII字符，而POST没有限制。
- GET比POST更不安全，因为参数直接暴露在URL上，所以不能用来传递敏感信息。
- GET参数通过URL传递，POST放在Request body中。
- (大多数)浏览器通常都会限制url长度在2K个字节，而(大多数)服务器最多处理64K大小的url
-  **GET产生一个TCP数据包;POST产生两个TCP数据包** 。 对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200(返回数据);而对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok(返回数据)。
#### http 不同请求的幂等性
- HTTP GET方法，用于获取资源，不管调用多少次接口，结果都不会改变，所以是幂等的。
- HTTP POST方法是一个非幂等方法，因为调用多次，都将产生新的资源。
- HTTP PUT方法直接把实体部分的数据替换到服务器的资源，我们多次调用它，只会产生一次影响,所以满足幂等性
- HTTP PATCH提供的实体需要根据程序或其它协议的定义，解析后在服务器上执行，以此来修改服务器上的资源。(换句话说，PATCH请求是会执行某个程序的，如果重复提交，程序可能执行多次，对服务器上的资源就可能造成额外的影响，这就可以解释它为什么是非幂等的了)
- HTTP DELETE方法用于删除资源，会将资源删除。调用一次和多次对资源产生影响是相同的，所以也满足幂等性
##
	var person1 = {
        name: 'person1',
        show1: function () {
          console.log(this.name)
        },
        show2: () => console.log(this.name),
        show3: function () {
          return function () {
            console.log(this.name)
          }
        },
        show4: function () {
          return () => console.log(this.name)
        }
      };
      var person2 = { name: 'person2' };

      person1.show1();   // person1
      person1.show1.call(person2);  // person2

      person1.show2();  // 'window'
      person1.show2.call(person2);  // 'window'

      person1.show3()();   // 'window'
      person1.show3().call(person2);  // person2
      person1.show3.call(person2)();   //  'window'

      person1.show4()();   // 'person1'
      person1.show4().call(person2);  // person1
      person1.show4.call(person2)();  // person2
##
	var a=0;
	function aa(a){
	    alert(a)
	    var a=3
	}
	aa(5)
	alert(a)
	//5,0   

	var a=0;
	function aa(){
	    alert(a)
	    var a=3
	}
	aa()
	alert(a)
	//undefined,0   
##js原型的继承
![图解构造器Function和Object的关系](http://oy99ekzhi.bkt.clouddn.com/js1.png)   
###一.简单原型链
拿父类实例来充当子类原型对象  
	function Super(){      // 父类
	    this.val = 1;
	    this.arr = [1];
	}
	function Sub(){			// 子类
	   
	}
	Sub.prototype = new Super();    // 核心 
###二.借用构造函数
	function Super(val){
	    this.val = val;
	    this.arr = [1];
	    this.fun = function(){  }
	}
	function Sub(val){
	    Super.call(this, val);   // 核心
	    // ...
	}
	//借父类的构造函数来增强子类实例，等于是把父类的实例属性复制了一份给子类实例装上了（完全没有用到原型）
###三.组合继承
	function Super(){
	    // 只在此处声明基本属性和引用属性
	    this.val = 1;
	    this.arr = [1];
	}
	//  在此处声明函数
	Super.prototype.fun1 = function(){};
	Super.prototype.fun2 = function(){};
	//Super.prototype.fun3...
	function Sub(){
	    Super.call(this);   // 核心
	    // ...
	}
	Sub.prototype = new Super();    // 核心
	// 把实例函数都放在原型对象上，以实现函数复用。同时还要保留借用构造函数方式的优点，通过Super.call(this);继承父类的基本属性和引用属性并保留能传参的优点；通过Sub.prototype = new Super();继承父类函数，实现函数复用
###四.寄生组合继承
	function beget(obj){   
	    var F = function(){};
	    F.prototype = obj;
	    return new F();
	}
	function Super(){
	    // 只在此处声明基本属性和引用属性
	    this.val = 1;
	    this.arr = [1];
	}
	//  在此处声明函数
	Super.prototype.fun1 = function(){};
	Super.prototype.fun2 = function(){};
	//Super.prototype.fun3...
	function Sub(){
	    Super.call(this);   // 核心
	    // ...
	}
	var proto = beget(Super.prototype); // 核心
	proto.constructor = Sub;            // 核心
	Sub.prototype = proto;              // 核心
	 
	var sub = new Sub();
	// 用beget(Super.prototype);切掉了原型对象上多余的那份父类实例属性
	// 改变子类的原型对象(将父类的原型对象复制给一个新对象,再将新对象赋给子类的原型对象),在子类的构造函数中,通过call借用父类的构造函数继承
	// beget个人觉得可以通过for in 循环向一个{}复制父类的原型对象
###五.原型式
	function beget(obj){   
	    var F = function(){};
	    F.prototype = obj;
	    return new F();
	}
	function Super(){
	    this.val = 1;
	    this.arr = [1];
	}
	 
	// 拿到父类对象
	var sup = new Super();
	// 生孩子
	var sub = beget(sup);  
###六.寄生式
	function beget(obj){   // 生孩子函数 beget：龙beget龙，凤beget凤。
	    var F = function(){};
	    F.prototype = obj;
	    return new F();
	}
	function Super(){
	    this.val = 1;
	    this.arr = [1];
	}
	function getSubObject(obj){
	    // 创建新对象
	    var clone = beget(obj); // 核心
	    // 增强
	    clone.attr1 = 1;
	    clone.attr2 = 2;
	    //clone.attr3...
	 
	    return clone;
	}
	 
	var sub = getSubObject(new Super());
##meta标签
meta标签是HTML中head头部的一个辅助性标签,可分为两大部分：http-equiv 和 name 属性。
   
http-equiv：相当于http的文件头作用，它可以向浏览器传回一些有用的信息，以帮助浏览器正确地显示网页内容。    
name属性：主要用于描述网页，与之对应的属性值为content，content中的内容主要是便于浏览器，搜索引擎等机器人识别，等等。

	<meta charset="utf-8">
	<meta http-equiv="x-ua-compatible" content="ie=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
	<!-- 以上 3 个 meta 标签 *必须* 放在 head 的最前面；其他任何的 head 内容必须在这些标签的 *后面* -->
 
	<!-- 允许控制资源的过度加载 -->
	<meta http-equiv="Content-Security-Policy" content="script-src 'self'; style-src nos.netease.com kaola.com;">
	上面的script-src代表脚本资源；style-src代表样式资源；'self'代表只信任当前域名下的外来资源，其他域下的资源全部会被拦截；nos.netease.com kaola.com代表信任nos.netease.com和kaola.com这两个域名下的资源。所以上面的标签的意义就是：对于脚本资源只信任本域下的，对于样式资源，除了本域还会加载nos.netease.com和kaola.com这两个域名下的。

	<!-- 尽早地放置在文档中 -->
	<!-- 仅应用于该标签下的内容 -->
 
	<!-- Web 应用的名称（仅当网站被用作为一个应用时才使用）-->
	<meta name="application-name" content="应用名称">
	 
	<!-- 针对页面的简短描述（限制 150 字符）-->
	<!-- 在*某些*情况下，该描述是被用作搜索结果展示片段的一部分 -->
	<meta name="description" content="一个页面描述">
	 
	<!-- 控制搜索引擎的抓取和索引行为 -->
	<meta name="robots" content="index,follow,noodp"><!-- 所有的搜索引擎 -->
	<meta name="googlebot" content="index,follow"><!-- 仅对 Google 有效 -->
	 
	<!-- 告诉 Google 不显示网站链接的搜索框 -->
	<meta name="google" content="nositelinkssearchbox">
	 
	<!-- 告诉 Google 不提供此页面的翻译 -->
	<meta name="google" content="notranslate">
	 
	<!-- 验证 Google 搜索控制台的所有权 -->
	<meta name="google-site-verification" content="verification_token">
 
	<!-- 用来命名软件或用于构建网页（如 - WordPress、Dreamweaver）-->
	<meta name="generator" content="program">
	 
	<!-- 关于你的网站主题的简短描述 -->
	<meta name="subject" content="你的网站主题">
	 
	<!-- 非常简短（少于 10 个字）的描述。主要用于学术论文。-->
	<meta name="abstract" content="">
	 
	<!-- 完整的域名或网址 -->
	<meta name="url" content="https://example.com/">
	 
	<meta name="directory" content="submission">
	 
	<!-- 基于网站内容给出一般的年龄分级 -->
	<meta name="rating" content="General">
	 
	<!-- 允许控制 referrer 信息如何传递 -->
	<meta name="referrer" content="never">
	 
	<!-- 禁用自动检测和格式化可能的电话号码 -->
	<meta name="format-detection" content="telephone=no">
	 
	<!-- 通过设置为 “off” 完全退出 DNS 预取 -->
	<meta http-equiv="x-dns-prefetch-control" content="off">
	 
	<!-- 在客户端存储 cookie，web 浏览器的客户端识别 -->
	<meta http-equiv="set-cookie" content="name=value; expires=date; path=url">
	 
	<!-- 指定要显示在一个特定框架中的页面 -->
	<meta http-equiv="Window-Target" content="_value">
	 
	<!-- 地理标签 -->
	<meta name="ICBM" content="latitude, longitude">
	<meta name="geo.position" content="latitude;longitude">
	<!-- 国家代码 (ISO 3166-1): 强制性, 州代码 (ISO 3166-2): 可选; 如 content="US" / content="US-NY" -->
	<meta name="geo.region" content="country[-state]">
	<!-- 如 content="New York City" -->
	<meta name="geo.placename" content="city/town">

viewport :

	<meta name ="viewport" content ="initial-scale=1, maximum-scale=3, minimum-scale=1, user-scalable=no"> <!-- `width=device-width` 会导致 iPhone 5 添加到主屏后以 WebApp 全屏模式打开页面时出现黑边 http://bigc.at/ios-webapp-viewport-meta.orz -->

content 参数:
***
- width viewport 宽度(数值/device-width)
- height viewport 高度(数值/device-height)
- initial-scale 初始缩放比例
- maximum-scale 最大缩放比例
- minimum-scale 最小缩放比例
- user-scalable 是否允许用户缩放(yes/no)

SEO 优化部分
***
	<meta name="keywords" content="your keywords">
	<meta name="description" content="your description">
	<meta name="author" content="author,email address">
	<meta name="robots" content="index,follow"> //定义网页搜索引擎索引方式，robotterms 是一组使用英文逗号「,」分割的值，通常有如下几种取值：none，noindex，nofollow，all，index和follow。
	<meta http-equiv="Cache-Control" content="no-siteapp" />  // 通过百度手机打开网页时，百度可能会对你的网页进行转码，脱下你的衣服，往你的身上贴狗皮膏药的广告，为此可在 head 内添加
##js中this
this是Javascript语言的一个关键字。 
es5它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。  
es6的箭头函数中的this:箭头函数没有自己的this, 它的this是继承而来; 默认指向在定义它时,它所处的对象(宿主对象),而不是执行时的对象, 定义它的时候,可能环境是window.
##js的执行环境
![EC的组成](http://oy985wqks.bkt.clouddn.com/1.png)  

- 变量对象（Variable object，VO）: 变量对象即包含变量的对象，除了我们无法访问它外，和普通对象没什么区别。
- [[Scope]]属性:作用域即变量对象，作用域链是一个由变量对象组成的带头结点的单向链表，其主要作用就是用来进行变量查找。而[[Scope]]属性是一个指向这个链表头节点的指针。作用域链其实就相当于一个变量对象的集合，其第一个元素是当前执行环境的变量对象，最后一个元素是全局执行环境的变量对象（在浏览器中即window对象）。
- this: 指向一个环境对象，注意是一个对象，而且是一个普通对象，而不是一个执行环境。


