#Buffer
buffer是一个类似于Array的对象,主要用于操作字节.buffer是一个典型的js和c++结合的模块,它将性能相关的部分用c++实现,将非性能的相关部分用js实现.buffer所占用的内存不是通过v8分配的,属于堆外内存.
##Buffer对象
buffer对象类似于数组,它的元素师16进制的两位数,即0到255的数值. 不同编码的字符串所占用的元素个数各不相同,如中文在UTF-8编码下占用3个元素,字母占一个. 
buffer同Array类似,可以访问length,可以通过下标访问元素,构造对象new Buffer(100) (括号里面指定buffer的长度)
##buffer内存分配
buffer对象的内存分配不是在v8的堆内存中,而是在ndoe的c++层实现申请.  
node采用slab分配机制(一种动态内存管理机制)来申请内存,slab有如下三种状态:

- full: 完全分配状态
- partial: 部分分配状态
- empty: 完全没有被分配状态
***
node以8kb作为界限区分buffer是大对象还是小对象,8kbu饿死每个slab的大小,在js层作为单位进行内存的分配.
###分配小对象
 如果指定buffer的小于8kb(按照小对象分配),分配中使用局部变量pool作为中间处理对象,处于分配状态的slab单元都指向它.

	var pool;
	function allocPool() {
		pool = new SlowBuffer(Buffer.poolSize); //Buffer.poolSize= 8*1024 (即8kb)
		pool.used = 0;
	}
![新构造的slab单元实例](http://oy99ekzhi.bkt.clouddn.com/15.png)此处的slab处于empty状态  
***
构造小对象的buffer: new Buffer(1024) ,在构造内部会先检查pool变量有没有被创建,如没有,则新创建:

	if (!pool || pool.length - pool.used < this.length) allocPool();
***
同时在new Buffer的过程中,当前buffer实例对象的parent属性指向该slab,记录从该slab上的开始的位置,slab自己也会同时记录被使用情况:

	this.parent = pool;
	this.offset = pool.used;
	pool.used += this.length;
	if (pool.used & 7) pool.used = (pool.used + 8) & ~7;
![从新的slab单元中初次分配buffer](http://oy99ekzhi.bkt.clouddn.com/16.png)此时slab状态是partial   
***
再次创建一个buffer对象是,会先判断这个slab的剩余空间是否足够,如果足够继续使用剩余空间,如果不够,则会新建slab存储,上一个slab中剩余的空间则会浪费
![在原slab中再分配一个buffer对象](http://oy99ekzhi.bkt.clouddn.com/17.png)  
同一个slab中可能分配给多个buffer对象使用,只有当前slab中所有的buffer对象在作用域释放并都可以回收,当前slab的8kb空间才会被回收,如有其中一个或者多个buffer对象无法被释放,那么整个slab的内存都不会释放
###分配大对象
如果需要8kb的buffer对象,将会直接分配一个slowBuffer对象(在c++中定义)作为slab单元,这个slab单元将会被这大buffer对象独占.
new Buffer(9*1024)的构造内部,会执行:  
	
	this.parent = new SlowBuffer(this.length);
	this.offset = 0;

##Buffer的转换
###字符串转Buffer
字符串转Buffer对象主要通过构造函数完成的: 
new Buffer(str,[encoding]) ,encoding是编码类型,默认是UTF-8,主要有:	
 
- ASCII
- UTF-8
- UTF-16LE/UCS-2
- Base64
- Binary
- Hex
###buffer的写入
buf.write(string,[offset],[length],[encoding]) 

- string: 写入buffer的字符串
- offset: 可选,buffer写入的索引值,默认是0
- length: 可选,写入的字节数,默认是buffer.length(string转成buffer实际长度)
- encoding: 可选,指定编码,默认是UTF-8
 
	buf = new Buffer(256);
	len = buf.write("www.runoob.com");  // 执行返回是写入buffer的实际大小,此例中是14

如果buffer空间不足,则只会写入部分字符串
###buffer转字符串
buf.toString([encoding],[start],[end]):  

- encoding:可选,默认UTF-8
- start:可选,指定读取索引位置,默认0
- end: 可选,指定读取结束位置,默认buf的末尾  
例如:
***
	buf = new Buffer(26);
	for (var i = 0 ; i < 26 ; i++) {
  		buf[i] = i + 97;
	}
	console.log( buf.toString('ascii'));       // 输出: abcdefghijklmnopqrstuvwxyz
	console.log( buf.toString('ascii',0,5));   // 输出: abcde
	console.log( buf.toString('utf8',0,5));    // 输出: abcde
	console.log( buf.toString(undefined,0,5)); // 使用 'utf8' 编码, 并输出: abcde`
###buffer不支持的编码类型
Buffer.isEncoding(encoding) 函数可以来判断node是否支持某个编码转换,返回布尔值. 在中国常见的**GBK,GB2321不在支持**的行列中.   
对于不支持的编码类型,借助第三方模块iconv和iconv-lite可以实现,iconv-lite采用纯js实现,iconv则通过c++调用libiconv库完成,前者比后者更轻量,无须编译和处理环境依赖直接使用.

	var iconv = require('iconv-lite');
	// Buffer转字符串
	var str = iconv.decode(buf, 'win1251');
	// 字符串转Buffer
	var buf = iconv.encode("Sample input string", 'win1251');

iconv和iconv-lite对无法转换的内容进行降级处理,iconv-lite无法转换的内容如果是多字节,会输出�,如果是单字节,则输出? . iconv有三级降级策略,会尝试翻译无法转换的内容,或者设置忽略这些内容.  
###buffer的拼接
buffer在使用场景中,通常是以一段一段的方式传输. 
	
	var fs = require('fs');
	var rs = fs.createReadStream('test.md');
	var data = '';
	rs.on("data", function (chunk){
		data += chunk;	
	});
	rs.on("end", function () {
		console.log(data);
	});
	


