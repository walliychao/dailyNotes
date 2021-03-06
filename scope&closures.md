### 编译原理

- 词法分析 Tokenizing
- 语法分析 生成语法分析树（AST）
- 代码生成

`JS是解释型语言，代码编译之后马上执行。`

### 编译器、执行引擎、作用域的关系

编译器解析代码，生成作用域。引擎查找作用域，并根据作用域执行代码。

`编译器声明变量和函数，引擎查找并执行赋值操作、垃圾回收。`

### LHS&RHS
- LHS: left hand search, 赋值操作符左侧查找，即引用查找
- RHS: right hand search, 赋值操作符右侧查找，即值查找

`js引擎跟作用域的交互基本可以归结为值查找和引用查找。`

### 词法作用域

作用域在函数及变量声明时就已经确定。

只有两种方法：eval, with可以改变词法作用域，但会导致**效率变差**。

- eval会在执行时动态改变代码的词法作用域（在eval内有声明语句）。strict模式下则只做用在eval内部，外部引用会出现引用错误。

- with(obj)会用obj创建一个新的作用域。使用时只能修改obj已有的属性值，修改不存在的属性时会泄露到global作用域中（strict模式不允许with）。

### 作用域生成

- 函数作用域

- 块作用域 (es6)

- 命名空间
  将某个对象或函数：global, object作为命名空间
  **js没有命名空间作用域**
  
#### 匿名函数的优缺点

优点：不污染外部命名空间，节省代码量。

缺点：

- 不易调试，在 call stack 里看不到函数名。
- 函数没法引用自身，arguments.callee已被淘汰。
- 影响代码可读性。


例外（块作用域）：

- with(obj): with后面会生成一个新的块作用域。

- try/catch: catch(err)后面会生成一个新的块作用域，在catch外部err引用非法。

- let/const: es6的变量声明都是块作用域内部有效的。

### 变量提升

``变量、函数声明会提升到（函数）作用域的顶部。``

只有函数声明会提升，函数表达式不会。且函数声明比变量声明更高。

重复的变量声明，之后的会被忽略；重复的函数声明会覆盖前面的声明。

### 闭包

  闭包就是一个函数能够在它的词法作用域之外记忆并且访问它的词法作用域内容。
  
  ``function return fn: 将函数引用传到作用域外``
  
#### 循环和闭包
  
- 添加匿名函数作为闭包
  
	```javascript
	    for (var i=1; i<=5; i++) {
	      (function(){
		var j = i;
		setTimeout( function timer(){
		  console.log( j );
		}, j*1000 );
	      })();
	    }
	```
	```javascript
	    for (var i=1; i<=5; i++) {
	      (function(j){
		setTimeout( function timer(){
		  console.log( j );
		}, j*1000 );
	      })( i );
	    }
	```
  
 - 使用**let**, 创造block作用域
  
	```javascript
	    for (var i=1; i<=5; i++) {
	      let j = i; // yay, block-scope for closure!
	      setTimeout( function timer(){
		console.log( j );
	      }, j*1000 );
	    }
	```
	```javascript
	  for (let i=1; i<=5; i++) {
	    setTimeout( function timer(){
	      console.log( i );
	    }, i*1000 );
	  }
	```

### js 模块(Module)

- 简单原型

	```javascript
	  function CoolModule() {
	    var something = "cool";
	    var another = [1, 2, 3];

	    function doSomething() {
	      console.log( something );
	    }

	    function doAnother() {
	      console.log( another.join( " ! " ) );
	    }

	    return {
	      doSomething: doSomething,
	      doAnother: doAnother
	    };
	  }

	  var foo = CoolModule();

	  foo.doSomething(); // cool
	  foo.doAnother(); // 1 ! 2 ! 3
	```
	构成模块的必要条件：

	    1. 要有一个外部的闭包函数，并且必须至少被执行一次（每次返回一个新的模块实例）。

	    2. 闭包函数必须至少返回一个内部函数，因此形成闭包，修改闭包函数内部的私有状态。
 
	单例模式的声明和执行方法：

	```javascript
	  var foo = (function CoolModule() {
	    var something = "cool";
	    var another = [1, 2, 3];

	    function doSomething() {
	      console.log( something );
	    }

	    function doAnother() {
	      console.log( another.join( " ! " ) );
	    }

	    return {
	      doSomething: doSomething,
	      doAnother: doAnother
	    };
	  })();

	  foo.doSomething(); // cool
	  foo.doAnother(); // 1 ! 2 ! 3
	```

	内部引用返回的api，方便之后动态的修改：

	```javascript
	  var foo = (function CoolModule(id) {
	    function change() {
	      // modifying the public API
	      publicAPI.identify = identify2;
	    }

	    function identify1() {
	      console.log( id );
	    }

	    function identify2() {
	      console.log( id.toUpperCase() );
	    }

	    var publicAPI = {
	      change: change,
	      identify: identify1
	    };

	    return publicAPI;
	  })( "foo module" );

	  foo.identify(); // foo module
	  foo.change();
	  foo.identify(); // FOO MODULE
	```

- 现代模块（类requirejs）

	requirejs demo:

	```javascript
	var MyModules = (function Manager() {
		var modules = {};

		function define(name, deps, impl) {
			for (var i=0; i<deps.length; i++) {
				deps[i] = modules[deps[i]];
			}
			modules[name] = impl.apply( impl, deps );
		}

		function get(name) {
			return modules[name];
		}

		return {
			define: define,
			get: get
		};
	})();
	```

	模块声明及执行方式：

	```javascript
	MyModules.define( "bar", [], function(){
		function hello(who) {
			return "Let me introduce: " + who;
		}

		return {
			hello: hello
		};
	} );

	MyModules.define( "foo", ["bar"], function(bar){
		var hungry = "hippo";

		function awesome() {
			console.log( bar.hello( hungry ).toUpperCase() );
		}

		return {
			awesome: awesome
		};
	} );

	var bar = MyModules.get( "bar" );
	var foo = MyModules.get( "foo" );

	console.log(
		bar.hello( "hippo" )
	); // Let me introduce: hippo

	foo.awesome(); // LET ME INTRODUCE: HIPPO
	```

- ES6 模块

	ES6把单个文件视为一个的模块，它并没有inline模式，模块**必须**定义在独立的文件中。

	**注意：**

	- 基于函数的JS模块是运行时的（编译器并不解析模块），即可以在定义后修改模块的API。

	- 相反的，ES6的模块是静态的（编译时解析、不能扩展），如果之后实例引用了定义时没有的API，会报引用错误。
