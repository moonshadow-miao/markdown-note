#混合
混合(mixins)是一种分发Vue组件中可复用功能的方式,混合对象可以包含任意组件选项.实现同一方法在多个组件之间复用
#选项合并
当组件和混合对象有相同方法时,   
如果是同名钩子函数都会被调用,且混合对象的钩子会先于组件的钩子调用.   

	var mixin = {
	  created: function () {
	    console.log('混合对象的钩子被调用')
	  }
	}
	new Vue({
	  mixins: [mixin],
	  created: function () {
	    console.log('组件钩子被调用')
	  }
	})
	// => "混合对象的钩子被调用"
	// => "组件钩子被调用"
   
如果是methods,components和directives,将会被混合同一个对象,如果重名,只会调用组件的部分,混合对象的重名部分不会调用.  

	var mixin = {
	  methods: {
	    foo: function () {
	      console.log('foo')
	    },
	    conflicting: function () {
	      console.log('from mixin')
	    }
	  }
	}
	var vm = new Vue({
	  mixins: [mixin],
	  methods: {
	    bar: function () {
	      console.log('bar')
	    },
	    conflicting: function () {
	      console.log('from self')
	    }
	  }
	})
	vm.foo() // => "foo"
	vm.bar() // => "bar"
	vm.conflicting() // => "from self"
##全局混合
使用全局混合对象的时候,将会影响到所有之后创建的Vue实例,合理使用.