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

### Destructuring 解构赋值
```
var [ a, b, c ] = foo();
var { x: x, y: y, z: z } = bar();
```
当参数名跟需要声明的名称完全一致时, 可以写成`var {x, y, z} = bar()`; 而需要声明的变量跟参数名不一致时
```
var { x: bam, y: baz, z: bap } = bar();
console.log( bam, baz, bap );		// 4 5 6
console.log( x, y, z );				// ReferenceError
```
可见`x:`代表原参数名, 后面表示声明的新变量; 跟普通的变量声明顺序相反

#### 先声明 后赋值
```
var a, b, c, x, y, z;

[a,b,c] = foo();
( { x, y, z } = bar() );
```
```
var o = {};
[o.a, o.b, o.c] = foo();
( { x: o.x, y: o.y, z: o.z } = bar() );

var a1 = [ 1, 2, 3 ],
	a2 = [];

[ a2[2], a2[0], a2[1] ] = a1;
```
```
var which = "x",
	o = {};

( { [which]: o[which] } = bar() );
```
可以先声明变量再使用; 也可以赋值给对象属性, 数组成员等复杂变量; 还可以使用计算属性

#### 重复赋值
```
var { a: X, a: Y } = { a: 1 };

X;	// 1
Y;	// 1
```
```
var { a: { x: X, x: Y }, a } = { a: { x: 1 } };

X;	// 1
Y;	// 1
a;	// { x: 1 }
```
同一个值可以重复赋值给不同的变量
```
p = { a, b, c } = o;
console.log( a, b, c );			// 1 2 3
p === o;						// true
```
连等时p与o指向同一个变量

#### 解构默认值
```
var [ a = 3, b = 6, c = 9, d = 12 ] = foo();
var { x = 5, y = 10, z = 15, w = 20 } = bar();

console.log( a, b, c, d );			// 1 2 3 12
console.log( x, y, z, w );			// 4 5 6 20
```
也可以跟普通赋值混合
```
var { x, y, z, w: WW = 20 } = bar();
console.log( x, y, z, WW );			// 4 5 6 20
```

#### 嵌套赋值
```
var [ a, [ b, c, d ], e ] = a1;
var { x: { y: { z: w } } } = o1;
```

#### 函数参数解构
```
function foo( { x, y } ) {
	console.log( x, y );
}
```

#### 解构默认值 + 函数参数默认值
```
function f6({ x = 10 } = {}, { y } = { y: 10 }) {
	console.log( x, y );
}

f6();						// 10 10
f6( undefined, undefined );			// 10 10
f6( {}, undefined );				// 10 10

f6( {}, {} );					// 10 undefined
f6( undefined, {} );				// 10 undefined

f6( { x: 2 }, { y: 3 } );			// 2 3
```
第一个参数是解构默认值, 传入`{}`时内部x为`undefined`, 默认值生效; 第二个参数是函数参数默认值, 传入参数是`{}`而非`undefined`, 默认值`{y: 10}`不会生效, 所以y是`undefined`

### super
super只允许在object的简写形式的方法中调用(`o = {a() {super.xxx()}}`), 且只允许`super.xxx`的调用形式

### Template string
template string 的`${}`中可以使用任何表达式, 如``to all of you ${upper( `${who}s` )}!``, 但最好不要使用多层template string

#### template的作用域
template类似一个立即执行函数(IIFE), 它的作用域也可以用IIFE解释(取template解析时的作用域下的变量):
```
function foo(str) {
	var name = "foo";
	console.log( str );
}

function bar() {
	var name = "bar";
	foo( `Hello from ${name}!` );
}

var name = "global";

bar();					// "Hello from bar!"
```

#### tagged template
类似一个方法, 后面的template string会作为处理成参数(strings, value)传入, tag甚至可以是一个返回一个新函数的函数调用: bar()`template ${foo}`
```
function foo(strings, ...values) {
	console.log( strings );
	console.log( values );
}

var desc = "awesome";

foo`Everything is ${desc}!`;
// [ "Everything is ", "!"]
// [ "awesome" ]
```

#### String.raw
String.raw是一个内置的tag, 可以返回string的raw形式, tag方法的参数strings.raw也能得到一个raw string
```
function showraw(strings, ...values) {
	console.log( strings );
	console.log( strings.raw );
}

showraw`Hello\nWorld`;
// [ "Hello
// World" ]
// [ "Hello\nWorld" ]

console.log( `Hello\nWorld` );
// Hello
// World

console.log( String.raw`Hello\nWorld` );
// Hello\nWorld

String.raw`Hello\nWorld`.length;
// 12
```

### arrow function
arrow function(=>)会自动把`this`置为当前作用域的this, 动态的this指向失效

因此仅在不需要`this`引用或者希望保持当前lexical的`this`(之前使用`var self = this`, 或者`.bind(this)`实现)的情况下才能使用箭头函数

**同时箭头函数的`arguments`, `super`, `new.target`都会lexical的继承当前作用域函数的值**
```自动
```
```
