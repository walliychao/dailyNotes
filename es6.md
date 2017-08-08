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
