# node.js笔记
##简介:
1. 诞生于2009年3月
2. 高性能、符合时间驱动、没有历史包袱三个原因,JavaScript成为Node的实现语言
***
##node的特点
- 异步IO
- 事件与回调函数:事件的编程方式轻量级,松耦合,只关心业务
- 单线程,javascript单线程的特点,不用像多线程那样,处处在意状态的同步问题,没有死锁的问题存在,也没有线程上下文交换所带来的性能上的开销. 缺点:没有利用多核cpu,错误会中断程序,大量计算会占用cpu导致无法继续调用异步I/O
- 跨平台,node基于libuv实现跨平台的架构
- node的应用场景,I/O密集型,高并发
***
##node的模块机制
### 一:commonJs
commonJs的愿景:希望javascript能够在任何地方运行
模块的引用:requrire("xxx") . 在模块中提供exports对象用于导出当前模块的方法或者变量,在模块中还存在一个**module**对象,**代表模块本身,而export是module的属性**
###二: node模块的实现
node引入模块,需要经历三个步骤:
1. 路径分析
2. 文件定位
3. 编译执行

模块分为两类: 一类是node提供的模块,称为核心模块;一类是用户编写的模块,称为文件模块.
####核心模块部分在node源代码编译的过程中,编译成二进制执行文件,在node进程启动时候,部分核心模块直接加载进内存,在路径分析中优先执行,且不需要文件定位再编译执行,所以加载速度最快
####文件模块运行时动态加载,需要完整的三个步骤,加载较慢
**node对引入的模块都会进行缓存,缓存的是编译和执行后的对象**,不论是核心模块还是文件模块,都是缓存优先

1. 模块标识符分析(路径分析): 核心模块(http,fs,path), ../ 或者 /xx/xx等按照路劲查找, 自定义模块(第三方包如express等) ,核心模块最优先加载,路劲形式按照确切的文件位置加载,速度次于核心模块. 自定义模块,按照制定的查找策略查找(生成一个路径数组,数组项分别是,当前文件目录下的node_modules目录,父目录下的node_modules目录,父目录的父目录下的node_modules目录,依次递增查询直到项目根目录)
2. 文件扩展名分析: node会按照.js,.json,.node的次序补足扩展名依次查询,如果加载的文件是.node或者.json文件,建议加上文件后缀
3. 不同的文件类型,编译载入方法不同: .js文件通过fs模块同步读取文件后编译执行, .node文件,用C/C++编写的扩展文件.通过dlopen()解析返回结果. .json文件,通过fs模块同步读取文件后,用JSON.parse()解析返回结果. 其余扩展名文件,他们被当做.js文件载入
4. 在编译的时候,node对获取的javascript文件进行头尾包装,在头部添加
###
	引入一个js文件,如:
	(function (exports, require, module, __filename, __dirname) {
		var math = require('math');  
		exports.area = function (radius) {  
		return Math.PI * radius * radius;  
		};
	});
###
这样每个模块文件之间都进行了作用域隔离,包装之后的代码会通过vm原生模块的runInThisContext()方法执行,返回一个具体的function对象.
####exports和module.exports的区别:
- module.exports初始值是一个空对象{}
- exports = module.exports ,exports指向module.exports对象的引用
- require()返回的是module.exports
javascript核心模块的定义,源文件模块通过process.binding('natives')取出,编译成功的模块缓存到NativeModule._cache对象上,文件模块则缓存到Module._cache对象上:
####
	function NativeModule(id) {
	this.filename = id + '.js';
	this.id = id;
	this.exports = {};
	this.loaded = false;
	}
	NativeModule._source = process.binding('natives');
	NativeModule._cache = {};
node内建模块,buffer,crypto,evals,fs,os通过C/C++编写,性能上优于js语言
## NPM 与 包
包其实是一个存档文件,即一个目录直接打包称为.zip或tar.gz格式的文件,安装后解压还原为目录.一个符合commonJs规范的包目录应该包含下列文件:

- package.json:包描述文件
- lib:用于存放javascript代码的目录
- bin:用于存放可执行二进制文件的目录
- doc:用于存放文档的目录
- test:用于存放单元测试用例的代码

commonJs为package.json文件定义了如下必需字段:

- name: 包名,需要小写的字母和数字组成,可以包含. _ 、-,但是不能有空给
- description 包简介
- version 版本号
- keywords 关键字**数组** npm中主要用来做分类搜索
- maintainers 包维护着列表
- contributors 贡献值列表
- bugs 一个可以反馈bug的url地址
- license 当前包使用的许可证列表
- repositories 托管源代码的位置列表
- **dependencies** 使用当前包所依赖的包列表
- **main** 引入包的时候,作为包其余模块的入口,如果不存在这个字段,会查找包目录下的index.js index.node index.json作为模块包入口
- **devdependencies**,一些模块只在开发时候需要依赖,应该放在这里

包的全局安装, -g ,全局模式并不是把一个模块包安为一个全局包的意思,并不意味着可以从任何地方通过require()引用到,实际上,-g是将一个包安装为全局可用的可执行命令,**根据包描述文件中bin字段配置,将实际脚本链接到与node可执行文件相同的路径下**, 事实上,通过全局安装的所有模块包都被安装进一个统一的目录下,这个目录可以通过 

	path.resolve(process.execPath, '..', '..', 'lib', 'node_modules'); ,如果node可执行文件的位置是/user/appData/local/node/bin/node,那么全局安装的模块目录就是/uer/appData/local/node_modules. 最后,通过软链接的方式将bin字段配置的可执行文件链接到node的可执行目录下

###AMD规范和CMD规范 ,用于浏览器的模块引用规范
 AMD规范是commonJs模块规范的一个延生,需要用到对应到库函数,requireJs,它的**模块定义如下:define(id?, dependencies?, factory)**; id:可选参数,用来定义模块的标识,如果没有提供参数,则用js文件名,dependencies:是一个当前模块依赖的模块数组,factory: 函数,函数最后需要return一个 exports对象 , 在页面上**加载模块:require([dependencies],function() {})** ,第二个参数回调函数会在第一个参数(依赖的模块数组)都加载完成后调用,且在回调中可以使用加载后的模块.

CMD规范由国内的玉伯提出,用到的库函数是sea.js,与AMD规范的主要区别在于定义模块和引入的部分,它的**模块定义: define(id,dependencies,function(require,exports,module){})** ,id:用文件名作为模块id,dependencies:一般不在define中写引入依赖,执行函数中传入三个参数,require引入依赖,exports导出对象,module是一个对象上面存储与当前模块相关联的一些属性和方法. 在页面上**加载模块,seaJs.use([dependencies],function(){}),参数同AMD**
####兼容多种模块规范
	!(function (name, definition) {
	// 检测上下文环境是否为AMD或CMD
		var hasDefine = typeof define === 'function',
		// 检查上下文环境是否为Node
		hasExports = typeof module !== 'undefined' && module.exports;
		if (hasDefine) {
		// AMD环境或CMD环境
		define(definition);
		} else if (hasExports) {
		// 定义为普通Node模块
		module.exports = definition();
		} else {
		// 将模块的执行结果挂在window变量中
		this[name] = definition();
		}
	})('hello', function () {
		var hello = function () {};
		return hello;
	});


	


 
