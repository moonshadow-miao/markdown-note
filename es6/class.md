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
##class表达式
与函数一样,类也可以使用表达式的形式定义

	const MyClass = class Me {
	  getClassName() {
	    return Me.name;
	  }
	};
	// 上面的代码使用表达式定义了一个类,这个类的名字是MyClass而不是Me,Me只能在class的内部代码可用,指代当前类.内部代码没用到的时候,可以省略这个Me. 
##不存在变量提升
类不存在变量的提升,必须要先定义后使用.

	new Foo(); // ReferenceError
	class Foo {}
##私有方法
ES6中不提供私有方法的定义,只能通过变通方法模拟
一种做法:在命名上加以区别  
 
	class Widget {
	  // 公有方法
	  foo (baz) {
	    this._bar(baz);
	  }
	  // 私有方法
	  _bar(baz) {
	    return this.snaf = baz;
	  }
	}
***
一种做法: 将私有方法移除模块,通过call或者apply调用  

	class Widget {
	  foo (baz) {
	    bar.call(this, baz);
	  }
	
	  // ...
	}
	
	function bar(baz) {
	  return this.snaf = baz;
	}
***
一种方法:利用Symbol值的唯一性,将私有方法的名字命名为一个Symbol值

	const bar = Symbol('bar');
	const snaf = Symbol('snaf');
	
	export default class myClass{
	  // 公有方法
	  foo(name) {
	    this[bar](name);
	  }
	  // 私有方法
	  [bar](name) {
	    return this[snaf] = name;
	  }
  	};
	// bar和snaf都是Symbol值，导致第三方无法获取到它们，因此达到了私有方法和私有属性的效果
##this的指向
类的方法内部如果含有this,它默认指向类的实例,但是,一旦单独使用该方法,可能会报错.

	class Logger {
	  printName(name = 'there') {
	    this.print(`Hello ${name}`);
	  }
	
	  print(text) {
	    console.log(text);
	  }
	}
	
	const logger = new Logger();
	const { printName } = logger;
	printName(); // TypeError: Cannot read property 'print' of undefined

解决方法: 
1.在构造函数方法中绑定this

	class Logger {
	  constructor() {
	    this.printName = this.printName.bind(this);  // 给printName方法绑定方法的调用者
	  }
	}
***
2.在构造函数中使用箭头函数
	
	class Logger {
	  constructor() {
	    this.printName = (name = 'there') => {
	      this.print(`Hello ${name}`);
	    };
	  }
	}
***
3.使用Proxy,获取方法的时候,自动绑定this

	function selfish (target) {
	  const cache = new WeakMap();
	  const handler = {
	    get (target, key) {
	      const value = Reflect.get(target, key);
	      if (typeof value !== 'function') {
	        return value;
	      }
	      if (!cache.has(value)) {
	        cache.set(value, value.bind(target));
	      }
	      return cache.get(value);
	    }
	  };
	  const proxy = new Proxy(target, handler);
	  return proxy;
	}
	
	const logger = selfish(new Logger());
##class的getter函数和setter函数
	class MyClass {
	  constructor() {
	    // ...
	  }
	  get prop() {
	    return 'getter';
	  }
	  set prop(value) {
	    console.log('setter: '+value);
	  }
	}
	
	let inst = new MyClass();
	
	inst.prop = 123;
	// setter: 123
	inst.prop
	// 'getter'
	// 上面代码中，prop属性有对应的存值函数和取值函数，因此赋值和读取行为都被自定义了。
##class的静态方法
类相当于实例的原型,所有在类中定义的方法,都会被实例继承,如果在一个方法前面,加上static关键字,就表示该方法不会被实例继承,而是只能直接通过类来调用.

	class Foo {
	  static classMethod() {
	    return 'hello';
	  }
	 static bar () {
	    this.baz();    // 调用的是静态方法,this指向Foo
	  }
	  static baz () {
	    console.log('hello');   // 静态方法可以与非静态方法重名。
	  }
	  baz () {
	    console.log('world');
	  }
	}
	
	Foo.classMethod() // 'hello'
	
	var foo = new Foo();
	foo.classMethod()
	// TypeError: foo.classMethod is not a function
	
	Foo.bar() // hello

父类的静态方法,可以被子类继承

	class Foo {
	  static classMethod() {
	    return 'hello';
	  }
	}
	
	class Bar extends Foo {
	}
	
	Bar.classMethod() 

静态方法也可以从super对象上调用

	class Foo {
	  static classMethod() {
	    return 'hello';
	  }
	}
	
	class Bar extends Foo {
	  static classMethod() {
	    return super.classMethod() + ', too';
	  }
	}
	
	Bar.classMethod() // "hello, too"
##Class的继承
	class ColorPoint extends Point {
	  constructor(x, y, color) {
	    super(x, y); // 调用父类的constructor(x, y)
	    this.color = color;
	  }
	
	  toString() {
	    return this.color + ' ' + super.toString(); // 调用父类的toString()
	  }
	}
	// super表示父类的构造函数,用来新建父类的this对象
	// 子类继承父类时,必须在constructor方法中调用super方法,因为子类没有自己的this对象,而是继承父类的this对象.如果不掉用super方法,子类得不到this对象
	// 如果子类没有定义constructor方法,这个方法会默认调用,如下:
	
	class ColorPoint extends Point {}
	// 等同于
	class ColorPoint extends Point {
	  constructor(...args) {
	    super(...args);
	  }
	}
***
在子类的构造函数中,只有调用super之后,才可以使用this关键字,否则会报错.
	
	class Point {
	  constructor(x, y) {
	    this.x = x;
	    this.y = y;
	  }
	}
	
	class ColorPoint extends Point {
	  constructor(x, y, color) {
	    this.color = color; // ReferenceError
	    super(x, y);
	    this.color = color; // 正确
	  }
	}
***
父类的静态方法，也会被子类继承
##Object.getPrototypeOf()
Object.getPrototypeOf方法可以用来从子类上获取父类,可以使用这个方法判断，一个类是否继承了另一个类。
##super()
super关键字可以当做函数使用(代表父类的构造函数),可以当做对象使用(在普通方法中，指向父类的原型对象；在静态方法中，指向父类).
###super作为函数使用
	class A {
	  constructor() {
	    console.log(new.target.name);
	  }
	}
	class B extends A {
	  constructor() {
	    super();  // 这里super虽然代表父类的构造函数,但是返回的是子类的实例,super内部的this指向B,这里super()相当于A.prototype.constructor.call(this)
	  }
	}
	new A() // A
	new B() // B
###super作为对象使用
	class A {
	  p() {
	    return 2;
	  }
	}
	
	class B extends A {
	  constructor() {
	    super();
	    console.log(super.p()); // 2
	  }
	}
	
	let b = new B();