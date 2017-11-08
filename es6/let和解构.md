###暂时性死区
	var tmp = 123;
	if (true) {
  		tmp = 'abc'; // ReferenceError
  		let tmp = '50';
	}  
只要块级作用域内存在let命令，它所声明的变量就“绑定”（binding）这个区域，不再受外部的影响.ES6明确规定，如果区块中存在let和const命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。这在语法上，称为“暂时性死区”.        
处于暂时性死区时候,typeof运行也会会抛出一个ReferenceError错误.但是一个变量根本没有被声明,typeof的结果是undefined,不会报错.  
例子:  

	例子一
	function bar(x = y, y = 2) {         // x=y 时报错,y此时还没声明 
  		return [x, y];
	}
	bar(); // 报错
	例子二:
	function f() { console.log('I am outside!'); }
	(function () {
  		if (false) {
    		// 重复声明一次函数f
    		function f() { console.log('I am inside!'); }   // 会
  		}
  	console.log(f)      // undefined
	}());
###const 
const实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动.复杂类型保存的是其内存地址.   
如果真的想将对象冻结，应该使用Object.freeze方法, 同理,如果对象中某个属性是复杂类型,同样保存的是内存地址.如果想完全深度的冻结对象,应该递归调用Object.freeze() ;
	
	var constantize = (obj) => {
    	Object.freeze(obj);
   		Object.keys(obj).forEach( (key, i)={
    		if ( typeof obj[key] === 'object' ) {
      		constantize( obj[key] );
    		}
  		});
	};
###变量的声明的六种方法
- var (es5),声明的全局变量,是顶层对象(window,global)的属性
- function (es5),声明的全局函数,是顶层对象(window,global)的属性
- let  和顶层对象属性无关
- const 和顶层对象属性无关
- import 和顶层对象属性无关
- class  和顶层对象属性无关
#解构
ES6中,允许按照一定模式,从数组和对象中提取值,对变量进行赋值,称为解构.
##数组的解构
	// 解构成功的例子
	let [foo, [[bar], baz]] = [1, [[2], 3]];    foo // 1,bar // 2,baz // 3
	let [ , , third] = ["foo", "bar", "baz"];  third // "baz"
	let [x, , y] = [1, 2, 3];         x // 1  ,y // 3
	let [head, ...tail] = [1, 2, 3, 4];     head // 1,tail /[2, 3, 4]
	let [x, y, ...z] = ['a'];  x // "a",y // undefined,z // []
	// 解构失败,变量就是undefined
	let [bar, foo] = [1];     foo//undefined
 	// 不完全解构,等号左边的模式只匹配一部分的等号右边的数组
	let [x, y] = [1, 2, 3];    x // 1 ,y // 2
	
	// 等号右边的值,如果不是数组,且转为对象后不具备iterator接口,解构报错
	let [foo] = {};       // 报错
	
	// 任何具有iterator接口的数据,都可以以数组形式解构
	function* fibs() {       // 斐波那契函数
  		let a = 0;
  		let b = 1;
  		while (true) {
    		yield a;
    		[a, b] = [b, a + b];
  		}
	}
	let [first, second, third, fourth, fifth, sixth] = fibs();    //sixth :5

## 数组解构的指定默认值
	let [foo = true] = [];     // foo: true
	let [x, y = 'b'] = ['a', undefined]; // x='a', y='b'
	let [x, y = 'b'] = ['a', null]; // x='a', y=null
	// ES6中使用 === 判断一个位置是否有值,
## 对象的解构
对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变量必须与属性同名，才能取到正确的值

	let { bar, foo } = { foo: "aaa", bar: "bbb" };   // bar:'bbb' , foo:'aaa'
	// 如果变量名与属性名不一致，必须写成下面这样
	let { foo: baz } = { foo: 'aaa', bar: 'bbb' };  // baz:'aaa'  
	// 实际上，对象的解构赋值是下面形式的简写
	let { foo: foo, bar: bar } = { foo: "aaa", bar: "bbb" };
	// 	嵌套赋值
	let obj = {};
	let arr = [];
	({ foo: obj.prop, bar: arr[0] } = { foo: 123, bar: true });    obj // {prop:123}  ,arr // [true]
	// 指定默认值
	var {x, y = 5} = {x: 1};    x//1 ,y//5
	
	// 如果要将一个已经声明的变量用于解构赋值，必须非常小心。
	let x;
	{x} = {x: 1};     // 报错,JavaScript 引擎会将{x}理解成一个代码块，从而发生语法错误,({x} = {x: 1});放在一个圆括号里面，就可以正确执行
	
##字符串的解构
字符串也可以解构赋值。这是因为此时，字符串被转换成了一个类似数组的对象。
	
	const [a, b, c, d, e] = 'hello';  a // "h",b // "e",c // "l",d // "l",e // "o"
	// 类似数组的对象都有一个length属性，因此还可以对这个属性解构赋值
	let {length : len} = 'hello';     len // 5
##数值和布尔值的解构赋值
解构赋值时，如果等号右边是数值和布尔值，则会先转为对象
##函数参数的解构
	[[1, 2], [3, 4]].map(([a, b]) => a + b);    // [3,7]
	// 函数参数的解构也可以使用默认值。
	function move({x = 0, y = 0} = {}) {
  		return [x, y];
	}
	move({x: 3, y: 8});     // [3, 8]
	move({x: 3}); // [3, 0]
	move({}); // [0, 0]
	move();   // [0, 0]
	
##解构的用途归纳
(1).交换变量的值
	
	let x = 1;
	let y = 2;
	[x, y] = [y, x]; 

(2) 从函数返回多个值

	function example() {
  		return {
    		foo: 1,
    		bar: 2
  		};
	}
	let { foo, bar } = example();

(3) 函数参数的定义

	function f({x, y, z}) { ... }
	f({z: 3, y: 2, x: 1});

(4) 提取JSON数据

	let jsonData = {
  		id: 42,
  		status: "OK",
  		data: [867, 5309]
	};
	let { id, status, data: number } = jsonData;

(5) 遍历Map结构 :Map结构原生支持Iterator接口，配合变量的解构赋值，获取键名和键值就非常方便。
	
	const map = new Map();
	map.set('first', 'hello');
	map.set('second', 'world');
	for (let [key, value] of map) {
  		console.log(key + " is " + value);
	}