#Iterator与generator
遍历器（Iterator）就是这样一种机制。它是一种接口，为各种不同的数据结构提供统一的访问机制。任何数据结构只要部署Iterator接口，就可以完成遍历操作（即依次处理该数据结构的所有成员）。  
Iterator 的作用有三个:

- 为各种数据结构，提供一个统一的、简便的访问接口
- 使得数据结构的成员能够按某种次序排列
- ES6创造了一种新的遍历命令for...of循环，Iterator接口主要供for...of消费

Iterator 的遍历过程:  

1. 创建一个指针对象，指向当前数据结构的起始位置。也就是说，遍历器对象本质上，就是一个指针对象
2. 第一次调用指针对象的next方法，可以将指针指向数据结构的第一个成员
3. 不断调用指针对象的next方法，直到它指向数据结构的结束位置
4. 每一次调用next方法，都会返回数据结构的当前成员的信息(具体来说，就是返回一个包含value和done两个属性的对象。其中，value属性是当前成员的值，done属性是一个布尔值，表示遍历是否结束。)
##默认Iterator接口
当使用for...of循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。  
一种数据结构只要部署了 Iterator 接口(默认的 Iterator 接口部署Symbol.iterator属性上)，我们就称这种数据结构是”可遍历的“（iterable）

	const obj = {
	  [Symbol.iterator] : function () {
	    return {
	      next: function () {
	        return {
	          value: 1,
	          done: true
	        };
	      }
	    };
	  }
	};

原生具备 Iterator 接口的数据结构如下。

- Array
- Map
- Set
- String
- TypedArray
- 函数的arguments对象
- NodeList对象
***
	 //在一个对象上部署遍历器:
	class RangeIterator {
	  constructor(start, stop) {
	    this.value = start;
	    this.stop = stop;
	  }
  	  [Symbol.iterator]() { return this; }
	  next() {
	    var value = this.value;
	    if (value < this.stop) {
	      this.value++;
	      return {done: false, value: value};
	    }
	    return {done: true, value: undefined};
	  }
	}

	function range(start, stop) {
	  return new RangeIterator(start, stop);
	}
	
	for (var value of range(0, 3)) {
	  console.log(value); // 0, 1, 2
	}
	// 上面代码是一个类部署 Iterator 接口的写法。Symbol.iterator属性对应一个函数，执行后返回当前对象的遍历器对象。
#generator
Generator函数是一个状态机,封装了多个内部状态,执行Generator函数会返回一个遍历器对象,可以依次遍历Generator函数内部的每一个状态.   
写法: function关键字与函数名之间有一个星号,函数体内部可以使用yield表达式.   
调用Generator函数后,函数不执行,返回的指向内部状态的指针对象,必须调用遍历器的next方法,指针向下移动.Generator函数是分段执行的,yield表达式是在暂停执行的标记,next方法可以恢复执行(每次执行都返回一个有value属性和done属性的对象,value是yiled表达式的值,done表示遍历是否结束),一直执行到return语句停止(最后next方法返回对象,value是return后面的表达式的值,done是true),如果没有retrun语句,则会执行完函数,返回一个对象(value是undefined,done是true).   
##yield表达式
遍历器对象next方法执行的时候,遇到yield表达式,就暂停执行后面的操作,并将紧跟在yield后面的表达式的值作为返回对象的value属性.    
yield后面的表达式,只有当调用next方法时,才会执行.  
ield表达式只能用在 Generator 函数里面.   
yield表达式如果用在另一个表达式之中，必须放在圆括号里面。

	function* demo() {
	  console.log('Hello' + yield); // SyntaxError
	  console.log('Hello' + yield 123); // SyntaxError
	
	  console.log('Hello' + (yield)); // OK
	  console.log('Hello' + (yield 123)); // OK
	}

yield表达式用作函数参数或放在赋值表达式的右边，可以不加括号。

	function* demo() {
	  foo(yield 'a', yield 'b'); // OK
	  let input = yield; // OK
	}
##next方法的参数
yield表达式本身没有返回值,但是next方法可以传入一个参数,该参数会被当做上一个yield表达式的返回值. 
 
