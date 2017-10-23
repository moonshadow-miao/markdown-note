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



