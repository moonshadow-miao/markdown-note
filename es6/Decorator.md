#类的修饰:
	@decorator
	class A {}
	
	// 等同于
	
	class A {}
	A = decorator(A) || A;
修饰器是一个对类进行处理的函数,修饰器函数的第一个参数,就是要被修饰的目标类. 注意，修饰器对类的行为的改变，是代码编译时发生的，而不是在运行时。这意味着，修饰器能在编译阶段运行代码。也就是说，修饰器本质就是编译时执行的函数。

添加实例属性，可以通过目标类的prototype对象操作。

	function testable(target) {
	  target.prototype.isTestable = true;
	}
	
	@testable
	class MyTestableClass {}