##node的I/O解决方案:
假设业务场景中有一组互不相干的任务需要完成,现行的主流方法有两种:

- 单线程串行依次执行
- 多线程并行完成

多线程的代价在于创建线程和执行期间线程上下文切换的开销较大,另外,在复杂的业务中,多线程编程经常面临死锁,状态同步问题. 优点是多线程在多核cpu上有效提升cpu的利用率.  

单线程顺序执行比较符合编程人员按照顺序思考的思维方式,易于表达.但是串行执行的缺点在于性能,I/O的进行会让后续任务等待,造成资源不能被更好的利用.  

node在两者直接给出它的方案:利用**单线程** ,远离多线程死锁,状态同步等问题;利用**异步I/O** ,让单线程远离阻塞,更好的使用CPU. 为了弥补单线程无法利用多核CPU的缺点,node提供了子进程,可以通过工作进程高校的利用CPU和I/O.
###异步I/O与非阻塞I/O
实际效果而言,异步和非阻塞都达到了并行I/O的目的,但是从计算机内核来说,其实是两回事. 操作系统内核对于I/O只有两种:阻塞和非阻塞.  
![调用阻塞I/O的过程](http://oy99ekzhi.bkt.clouddn.com/1.png)

内核在进行文件阻塞I/O操作时,**需要先打开文件描述符,然后根据文件描述符去实现文件的数据读写**,非阻塞不带数据立即返回,CPU可以继续用来处理其他事物,然后通过轮询和回调技术,根据文件描述符再次读取数据返回给应用程序.  
![调用非阻塞的过程](http://oy99ekzhi.bkt.clouddn.com/2.jpg)

####理想的非阻塞异步I/O:
应用程序发起非阻塞调用,无须通过遍历或者事件唤醒等方式轮询,直接可以处理下一个任务,只需在I/O完成后通过信号或回调将数据传递给应用程序.如图:  
 ![理想的异步I/O调用](http://oy99ekzhi.bkt.clouddn.com/3.jpg)
####现实的异步I/O:
通过让部分的线程进行阻塞I/O或者非阻塞I/O加轮询技术来完成数据获取(线程池),让一个线程进行计算处理,通过线程之间的通信将I/O得到的数据进行传递,实现异步I/O,如图:  
![现实的异步I/O调用](http://oy99ekzhi.bkt.clouddn.com/4.jpg)  

由于window平台和*nix平台的差异,node提供了libuv作为抽象封装层,兼容所有平台,node在编译期间会判断平台条件,选择性编译unix目录或是win目录下的源文件.如图:  
![基于libuv的架构](http://oy99ekzhi.bkt.clouddn.com/5.jpg)
###node的异步I/O:
事件循环:进行启动时,node会创建一个循环,每执行一次循环过程称为Tick,每个Tick的过程就是查看是否有事件待处理,如有,取出事件并执行相关的回调函数.然后进入下个循环,如果没有事件处理,退出进程.如:  
![事件循环](http://oy99ekzhi.bkt.clouddn.com/6.png)  

观察者:  
每个事件循环中有一个或者多个观察者,而判断是否有事件要处理的过程就是向这些观察者询问是否有要处理的事件. 在node中事件主要来源于网络请求,文件I/O等,对应有文件I/O观察者,网络I/O观察者.  

生成请求对象:  

	fs.open = function(path, flags, mode, callback) {
		binding.open(pathModule._makeLong(path), stringToFlags(flags), mode, callback);
	};  
 
fs.open()的作用是根据指定的路径和参数去打开一个文件,从而得到一个文件描述符.具体是,js调用node的核心模块fs,核心模块调用c++内建模块,内建模块通过libuv进行系统调用(判断平台,调用当前平台下的方法)uv_fs_open()方法,且创建了FSReqWrap**请求对象** ,从js层传入的参数和当前方法的都会被封装在这个对象上,如从js层传入的回调函数就会被存储在对象的**oncomplete_sys**属性上. 然后调用QueueUserWorkItem()方法将这个对象推入线程池中执行,js调用立即返回,继续执行后续操作.  
请求对象是异步I/O过程中重要的中间产物,**所有的状态都保存在这个对象中,包括送入线程池等待执行以及I/O操作完毕后的回调处理** .

执行回调:  
线程池中的I/O操作调用完毕后,会将获取的结果存储在req->result属性上,PostQueuedCompletionStatus()向IOCP提交执行状态,并将线程归还线程池.每次的循环Tick中通过GetQueuedCompletionStatus()获取提交的状态判断有执行完的I/O,把请求对象加入I/O观察者队列中,**I/O观察者会取出请求对象的result属性作为参数,取出oncomplate_sym属性上面的回调执行** ,如图:  
![异步I/O的流程](http://oy99ekzhi.bkt.clouddn.com/7.png)  
**事件循环,观察者,请求对象,I/O线程池这四者共同构成node异步I/O模型的基本要素**

####非I/O的异步API:
setTimeout(),setInterval(),setImmediate(),process.nextTick() 

 **定时器** : 执行setTimeout(),setInterval()的时候,会被插入到定时器观察者内容的一个红黑树种,每次Tick执行时,会从该红黑树种迭代取出定时器对象,检查是否超过定时时间,如是,形成一个事件且调用定时器的回调函数.如图:  
 ![setTimeout的执行](http://oy99ekzhi.bkt.clouddn.com/eigtht.png)

**process.nextTick()** :每次调用process.nextTick()方法,会将回调函数放入队列中,在下一轮tick时取出执行

**setImmediate()** : setImmediate()和process.nextTick()方法十分类似,都是将回调函数延迟执行.但是执行的优先级低于process.nextTick(),process.nextTick()属于idle观察者,setImmediate()属于check观察者,在每轮循环检查中,idle观察者先于I/O观察者,I/O观察者有限于check观察者.**process.nextTick()的回调函数保存在一个数组中,setImmediate()的结果则是保存在链表中,process.nextTick()在每轮循环中会将数组中的回调函数全部执行完,而setImmediate()在每轮循环中执行链表中的一个回调函数** ,例如:  

	// 加入两个nextTick()的回调函数
	process.nextTick(function () {
		console.log('nextTick延迟执行1');
	});
	process.nextTick(function () {
		console.log('nextTick延迟执行2');
	});
	// 加入两个setImmediate()的回调函数
	setImmediate(function () {	
		console.log('setImmediate延迟执行1');
	// 进入下次循环
	process.nextTick(function () {
		console.log('强势插入');
	});
	});
	setImmediate(function () {
	console.log('setImmediate延迟执行2');
	});
	console.log('正常执行');
	执行结果是:
	
	正常执行
	nextTick延迟执行1
	nextTick延迟执行2
	setImmediate延迟执行1
	强势插入
	setImmediate延迟执行2

###利用node构建高性能服务器:
![利用node构建web服务器](http://oy99ekzhi.bkt.clouddn.com/9.png)
	
#异步编程
**高阶函数** :高阶函数可以把函数作为参数,或是将函数作为返回值的函数 
##难点:  
1. 异常处理: 异步I/O的实现主要包括两个阶段,提交请求和处理结果,异步方法通常在提交请求时直接返回,这个时候无法捕获后面阶段的异常(如:回调执行时候抛出的异常).所以, node在处理异常上形成一种约定,将异常作为回调函数的第一个实参传回,如果是空值,表明异步调用没有异常抛出.
2. 函数嵌套过深
3. 没有阻塞代码:sleep()线程沉睡代码在js中没有,可以setTimeout替代
4. 多线程编程
5. 异步转同步,node中也会提供部分同步API

##异步编程解决方案
###事件发布/订阅模式
node自身提供的events模块,具有addListener()/on() ,once(), removeListener() ,removeAllListener(),emit().等方法.示例:

	// 订阅
	emitter.on("event1", function (message) {
		console.log(message);
	});
	// 发布
	emitter.emit('event1', "I am message!");
发布/订阅模式可以实现一个事件和多个回调函数的关联,**这些回调函数又称为事件侦听器** .这种模式常常用来解耦业务逻辑,事件发布者无须关注订阅的侦听器如何实现业务逻辑,不用关注多少个侦听器存在,数据通过消息的方式灵活的传递.  
在一些场景中,将不变的部分封装在组件中,将变化的部分暴露给外部处理,此处事件订阅的设计就是组件的接口设计.  
从另一个角度看,事件侦听模式也是一种钩子函数,利用钩子导出内部数据或者状态给外部,不用关注组件如何启动和执行,只需关注需要事件点(钩子函数触发).  
	
	var options = {
		host: 'www.google.com',port: 80,path: '/upload',method: 'POST'
	};
	var req = http.request(options, function (res) {
		console.log('STATUS: ' + res.statusCode);
		console.log('HEADERS: ' + JSON.stringify(res.headers));
		res.setEncoding('utf8');
		res.on('data', function (chunk) {
		console.log('BODY: ' + chunk);
	});
	res.on('end', function () {
		// TODO
		});
	});
	req.on('error', function (e) {
		console.log('problem with request: ' + e.message);
	});
	req.write('data\n');
	req.end();
上述代码中,我们只需要关心error,data,end这些事件如何处理,无须关心内部的流程.
node对事件发布/订阅的机制做了额外的处理:

- 如果对一个事件添加了超过10个侦听器,将会得到一条警告,可以调用setMaxListeners(0)移除警告.
- 为了处理异常,EventEmitter对象检查是否有error事件侦听,如有则将错误抛给侦听器处理,否则抛出异常导致线程退出.
***
	var proxy = new events.EventEmitter();
	var status = "ready";
	var select = function (callback) {
		proxy.once("selected", callback);
		if (status === "ready") {
		status = "pending";
		db.select("SQL", function (results) {
			proxy.emit("selected", results);
			status = "ready";
			});
		}
	};
	这里多次调用select()时,利用once方法将所有的请求回调压入事件队列总,利用其执行一次就会将监视器移除的特点,保证每一个回调只会被执行一次.对于相同的sql语句,同一时间只会有一个查询执行,一旦查询结束,得到的结果可以被所有的相同调用共用.
***
###多异步之间的协作方案:
异步编程中,也会出现事件和侦听器的关系是多对一的情况,一个业务逻辑可能依赖多个回调执行后的结果.   
业务场景例子: 模版读取 + 数据读取 + 本地资源读取 => 渲染页面    

	var count = 0;
	var results = {};
	var done = function (key, value) {
		results[key] = value;
		count++;
		if (count === 3) {
		// 渲染页面
		render(results);
		}
	};
	fs.readFile(template_path, "utf8", function (err, template){
		done("template", template);
	});
	db.query(sql, function (err, data) {
		done("data", data);
	});
	l10n.get(function (err, resources) {
		done("resources", resources);
	});
	// 借助哨兵变量(此处用来检测次数的变量)处理异步协作的结果
	
	var after = function (times, callback) {
		var count = 0, results = {};
		return function (key, value) {
					results[key] = value;
					count++;
					if (count === times) {
					callback(results);
					}
				};
	};
	var done = after(times, render);
	var emitter = new events.Emitter();
	emitter.on("done", done);
	fs.readFile(template_path, "utf8", function (err, template) {
		emitter.emit("done", "template", template);
	});
	db.query(sql, function (err, data) {
		emitter.emit("done", "data", data);
	});
	l10n.get(function (err, resources) {
	emitter.emit("done", "resources", resources);
	});
	// 借助偏函数处理哨兵变量和第三方函数的关系

###promise/deferred模式
通过node的events模块完成promiss的实现

	var Promise = function(){
		EventEmitter.call(this);
	}
	Util.inherits(Promise,EventEmitter);
	Promise.prototype.then = function(fulfilledHandler, errorHandler, progressHandler) {
		if(typeof fulfilledHandler === 'function'){
			// 利用once方法,保证成功回调只执行一次
			this.once('success',fulfilledHandler);
		}
		if(typeof errorHandler === 'function'){
			this.once('error',errorHandler);
		}
		if(typeof progressHandler === 'function'){
			this.once('progress',progressHandler);
		}
		return this
	};
	var Deferred = function(){
		this.state = 'unfulfilled';
		this.promise = new Promise();
	};
	Deferred.prototype.resolve = function (obj){
		this.state = 'fulfilled';
		this.promise.emit('success',obj);
	};
	Deferred.prototype.reject = function (obj){
		this.state = 'failed';
		this.promise.emit('error',obj);
	};
	Deferred.prototype.progress = function (obj){
		this.promise.emit('progress',obj);
	}


	


	




