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