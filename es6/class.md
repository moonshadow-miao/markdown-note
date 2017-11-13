#class
类的内部所有定义的方法，都是不可枚举的（non-enumerable）

	class Point {
	  constructor(x, y) {
	    // ...
	  }
	
	  toString() {
	    // ...
	  }
	}
	
	Object.keys(Point.prototype)
	// []

类的属性名，可以采用表达式。  
	
	let methodName = 'getArea';

	class Square {
	  constructor(length) {
	    // ...
	  }
	
	  [methodName]() {
	    // ...
	  }
	}
##constructor方法
constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加。  
constructor方法默认返回实例对象（即this），完全可以指定返回另外一个对象

	class Foo {
	  constructor() {
	    return Object.create(null);
	  }
	}
	
	new Foo() instanceof Foo  // false
##类的实例对象
类必须使用new调用，否则会报错。这是它跟普通构造函数的一个主要区别，后者不用new也可以执行

	class Point {
	
	  constructor(x, y) {
	    this.x = x;
	    this.y = y;
	  }
	
	  toString() {
	    return '(' + this.x + ', ' + this.y + ')';
	  }
	
	}
	
	var point = new Point(2, 3);
	
	point.toString() // (2, 3)
	
	point.hasOwnProperty('x') // true
	point.hasOwnProperty('y') // true
	point.hasOwnProperty('toString') // false
	point.__proto__.hasOwnProperty('toString') // true