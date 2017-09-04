### 块作用域

let, const 声明的变量只在当前块作用域下有效, 提前使用还未声明的变量会报`Reference Error`

- 用`typeof`测试一个未声明变量时也会报错 
```
{
	// `a` is not declared
	if (typeof a === "undefined") {
		console.log( "cool" );
	}
	// `b` is declared, but in its TDZ
	if (typeof b === "undefined") {		// ReferenceError!
		// ..
	}
	let b;
}
```

- `const`声明的变量a不可变(不可重新指定其它值), 但a指向的数组或对象是可变的
```
{
	const a = [1,2,3];
	a.push( 4 );
	console.log( a );		// [1,2,3,4]
	a = 42;					// TypeError!
}
```

- 块作用域函数: 在ES6中, 所有在代码块中定义的函数都只存在于块作用域中
```
if (something) {
	function foo() {
		console.log( "1" );
	}
}
else {
	function foo() {
		console.log( "2" );
	}
}
foo();		// ??
```

在ES6之前的环境中, 不管something是什么结果都会是2, 因为第二次声明的函数foo覆盖了第一次的;
而在ES6环境中则会报ReferenceError错误, 因为foo在块作用域之外不存在

### Spread/Rest
```
function foo(...args) {  // rest, gather
	console.log( args ); // spread
}
```

### function默认参数
```
function foo(x = 11, y = 31) {
	console.log( x + y );
}
```
- x, y传`undefined`时相当于未传值, 使用默认值
- rest参数无法指定默认值

#### default value expression
```
function foo(x = y + 3, z = bar( x )) {
	console.log( x, z );
}
```
默认参数也可以是表达式, 包括函数; 只有在参数未传或是`undefined`的时候表达式才会实际执行

表达式中使用的参数是在特殊作用域中的(`foo(...)`中), 并不在函数体(function body)中
```
var w = 1, z = 2;

function foo( x = w + 1, y = x + 1, z = z + 1 ) {
	console.log( x, y, z );
}

foo();					// ReferenceError
```
其中`x = w + 1`会找到外层的w声明, 成功执行; `y = x + 1`会使用之前声明并赋值的x, 成功执行; `z = z + 1`因为z已经声明, 所以不会再到外层作用域寻找, 又因为使用z的时候z还没有声明(类似let声明), 因此会报错
