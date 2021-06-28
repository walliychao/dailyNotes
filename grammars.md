### -Function
```javascript
function a(b, c) {}
a.length    // 2
```
函数对象的length属性表示它定义时参数的个数

### -typeof
```javascript
typeof null === 'object'
typeof NaN === 'number'
```

```javascript
var a
typeof a === 'undefined'
typeof b === 'undefined'
typeof undefined === 'undefined'
```
未声明和未赋值的变量typeof都是undefined; `undefined`的typeof也是undefined

### -hoist
```javascript
if (0) {
    var a = 3;
}
a; // undefined
```

```javascript
if (0) {
    a = 3;
}
a; // referenceError
```

如果用var定义, 即使未执行if内的语句, 也会进行变量提升

用var声明属于显示声明, 会在预编译阶段创建一个全局或局部变量, 已声明的变量不可删除, 使用let, const声明的变量不可重复声明, 否则会报错

未用var声明直接赋值属于隐式声明, 实际上会查找当前作用域链, 找到对应属性则进行修改, 找不到时会在最上层(window)对象上创建属性并赋值;因此在执行阶段才存在

### -`undefined` vs. `null` vs. `void 0`

undefined和null本身是一个基本数据类型, 只有一个值就是自身; void是一个运算符, 可以使任何表达式返回undefined

**undefined**: 是一个内置的变量, 默认值就是undefined; typeof undefined === 'undefined'

表示声明未赋值, 函数未返回值或未传参时默认返回, 可以作为局部变量被重写(var undefined = '123')

在non-strict模式下可以改变全局的undefined值, strict模式下不可以
```javascript
function foo() {
	undefined = 2; // really bad idea!
}
```

在strict和non-strict模式下都可以改变本地的undefined值
```javascript
function foo() {
	"use strict";
	var undefined = 2;
	console.log( undefined ); // 2
}
```
**绝对不要改undefined值!!**

**null**: 表示一个空对象指针, typeof null === 'object'

在需要解除一个变量的对象引用时使用, 下次垃圾回收时该变量所占用内存可以被回收

*console.log 的对象不会被垃圾回收*

**void 0**: void运算符对给定的表达式进行求值，然后返回undefined(可以把任何表达式的值变成undefined返回);

因为undefined可以被重写, 所以希望返回原始值undefined的地方可以赋值 void 0

```javascript
void 0      // undefined
void 3+4    // undefined
```

### -Array
```javascript
a["13"] = 42;
a.length; // 14
```

注意不要形成sparse array, `delete`, `Array(3)`都会形成sparse array

ES6的`Array.from({length: 3})`可以生成一个每个元素都有值(undefined)的数组; ES5形式`Array.apply(null, {length: 3})`也有同样效果

### -Number
- |: `a | 0`可以把a转成一个32bit的数字: 去掉小数位, null | NaN | Infinity 转成0, 所有bitwise operation都会首先做To32Bit操作

- ~: `~(-1) => 0`, ~x === -(x + 1)

- NaN: `NaN !== NaN` NaN是唯一一个不等于它自己的值

- Infinity: 只能从普通值计算得到Infinity, 而不能从Infinity计算得到普通值; 即Infinity跟任何值计算都是Infinity, 除了 `Infinity / Infinity === NaN`, `a / Infinity === 0`等

- -0: 乘除运算可能结果会为-0
	```javascript
	var a = 0 / -3                  // -0

	// -0 stringify之后会变成0
	a.toString();			// "0"
	a + "";				// "0"
	String( a );			// "0"
	JSON.stringify( a );		// "0"

	// '-0'转成数字后是-0
	+"-0";				// -0
	Number( "-0" );			// -0
	JSON.parse( "-0" );		// -0

	// -0与0全等...
	-0 == 0;	// true
	-0 === 0;	// true
	0 > -0;		// false

	// 识别-0的方法
	(n === 0) && (1 / n === -Infinity)
	```
- Object.is: ES6的Object.is()方法可以判断一个变量是否是NaN 或 -0

### -值&引用
所有的原始值都是值传递, 对象都是引用传递; 原始值的对象包装(boxed value)是引用传递, 但是它包含的原始值是不可变的, 因此跟原始值相同的效果

### -类型转换
一个对象转换成特定的原始值类型时, 会首先检查是否有`valueOf`方法, 有则调用这个方法; 否则检查是否有`toString`方法将对象转化为原始值之后再做类型转换; 如果两个方法都没有, 则会报`TypeError`

- Date to Number
	```javascript
	var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );
	+d; // 1408369986000
	```
- Symbol
	```javascript
	var s1 = Symbol( "cool" );
	String( s1 );					// "Symbol(cool)"
	s1 + "";						// TypeError
	```

Symbol可以显示转化为string, 但不能隐式转; 不可以转成Number, 会报错; 可以转成Boolean(true)

### -falsy value
- undefined
- null
- false
- +0, -0, and NaN
- ""

除了这些值其它的值都会被转换为true

### -JSON stringify
JSON.stringify无法转换undefined, function, Symbol和有循环引用的对象; 处理有循环引用的对象时会报错, 其它值如果直接转换会返回undefined, 如果在数组中会替换为null, 如果在Object中则会忽略这个属性

如果一个对象有`toJSON`这个方法, 则会默认调用这个方法得到*一个JSON-safe的值之后*再进行JSON stringify转换, 可以重写这个方法来改变转换时使用的实际对象

stringify的第二个参数可以传一个replacer, 跟toJSON同样的功能, 可以过滤json转换时用到的属性

- replacer如果是一个数组, 则包含所有应该被转换的属性名, 没有包含的属性会被忽略
- replacer如果是一个方法, 会对要转换的object和它的所有属性执行这个方法, 参数是key和value, 如果要过滤某个属性则返回undefined, 否则返回对应的value值

stringify的第三个参数是indentation设置, 如果是数字则表示用几个空格, 如果是字符串(如'----')则用字符串做indentation

### -parseInt
如果传给parseInt的值不是string, 则会默认转成string后再parse成number
```javascript
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );		// 16
parseInt( "103", 2 );		// 2
```

### -双等号==
- ==两边的变量都是object(function, array)时, 只有当变量指向的对象完全一致(**引用一致**)时才会返回true, 而且跟===的表现一样, 不会做类型转换
	```javascript
	var aa = [1,2,3]
	var bb = [1,2,3]
	aa == bb              // false
	aa === bb             // false

	var aa = [1,2,3]
	var bb = aa 
	aa == bb              // true
	aa === bb             // true
	```
	
- string <==> number
string跟number比较会把string转成number形式再进行比较

- anything <==> boolean
任何值跟boolean比较时, 都会把boolean先转成number(0或1)再进行比较

- null <==> undefined => true
null和undefined是互相==的, 任何其它值(false, '', 0)都不与null或undefined ==

- object <==> non-object
object在与非object比较时, object会通过`valueOf`或`toString`转成一个原始值, 转成原始值之后会跟non-object参照前三条进行比较; 一个原始值与它boxed之后的对象是 == 的

**Object -> non-Object, boolean -> number, string -> number**

**首先把值转换成原始类型, 如果包含null或undefined则按第三条比较; 如果类型相同则直接比较; 如果类型不同则转换成number比较**

```javascript
false == "0";			// true
false == 0;			// true
false == "";			// true
false == [];			// true
"" == 0;			// true
"" == [];			// true
0 == [];			// true
```

**666**
```javascript
[] == ![]			// true -- ![]会转成false再比较
2 == [2];			// true -- [2]会转成'2'
"" == [null];			// true -- [null]会转成''
"" == [undefined];		// true -- [undefined]会转成''

0 == "\n";			// true -- '\n', ' ', '\r', '\t'等*空字符串*都会转成数字0比较
```

**==两边如果可能出现`true`, `false`, 绝对不要使用; 如果可能出现`0`, `''`, `[]`, 尽量避免使用**

### - <, >, <=, >=
`a > b`会被处理成`b < a`; `a >= b`会被处理成`b <= a`; `b <= a`实际上是`!(b > a)`即`!(a < b)`

```javascript
var a = { b: 42 };
var b = { b: 43 };

a < b;	// false, 两边都转成'[object Object]'
a == b;	// false, 两边都是object, 比较reference
a > b;	// false, 同a < b

a <= b;	// true, 因为a > b 为fasle
a >= b;	// true, 因为a < b 为fasle
```

比较时首先将两边转换成原始值, 如果一边存在null或undefined则直接返回false, 如果两边都是string则按字典顺序比较, 如果类型不同则转换成数字进行比较

### -quirks
- 连等问题
	```javascript
	var a = 3
	a = b = a++
	a // 3
	b // 3
	```
	b没有声明; a被赋值为`b = a++`这个表达式返回的值, 实际是b的值

- block
	```javascript
	[] + {}; 			// "[object Object]"
	{} + []; 			// 0
	```
	第二行会被解析成一个独立的空block`{}`, 再加一个表达式`+[]`, `+[]`的值是0, 所以返回0
	
- function parameter
	```javascript
	var b = 3;
	function foo( a = 42, b = a + b + 5 ) {
		// Uncaught ReferenceError: b is not defined
	}
	```
	error, ES6的函数参数默认值是let声明的, `b = a+b+5`在使用b时是没有声明的; 函数内部不可以再用let声明相同的变量, 但可以用var重复声明, 且可以正常传值
	
	```javascript
	function foo(arg1) {
	   let arg1  // Uncaught SyntaxError: Identifier 'arg1' has already been declared
	}
	
	function foo(arg1) {
	   var arg1  // 'arg1'
	}

	foo('arg1')
	```
	
	参数传undefined时跟没有传参一样效果, 会使用默认值, 但arguments对象长度会有差别
	
	```javascript
	function foo(a) {
		a = 42;
		console.log( arguments[0] );
	}

	foo( 2 );	// 42 (linked)
	foo();		// undefined (not linked)
	```
	在函数中, arguments中的值和参数名是互相链接的, 改变一个会同时改变另一个的值, 不传参时不会链接
	
	```javascript
	function foo(a) {
		"use strict";
		a = 42;
		console.log( arguments[0] );
	}

	foo( 2 );	// 2 (not linked)
	foo();		// undefined (not linked)
	```
	在strict模式下这种链接又不存在...
	
	**不要同时使用arguments和参数名两种方式, 特别是在修改参数值的情况下**
	
 - finally
	`try...catch...finally`中finally中如果有特殊代码, 可能发生意外操作

	```javascript
	function foo() {
		try {
			return 42;
		}
		finally {
			throw "Oops!";
		}

		console.log( "never runs" );
	}

	console.log( foo() );
	// Uncaught Exception: Oops!
	```
	
	`try`正常执行, `finally`抛出错误, 最终会抛错


	```javascript
	function foo() {
		try {
			return 42;
		}
		finally {
			// no `return ..` here, so no override
		}
	}

	function bar() {
		try {
			return 42;
		}
		finally {
			// override previous `return 42`
			return;
		}
	}

	function baz() {
		try {
			return 42;
		}
		finally {
			// override previous `return 42`
			return "Hello";
		}
	}

	foo();	// 42
	bar();	// undefined
	baz();	// "Hello"
	```
	
	finally和try中如果都有return, finally会覆盖try的return


	```javascript
	function foo() {
		bar: {
			try {
				return 42;
			}
			finally {
				// break out of `bar` labeled block
				break bar;
			}
		}

		console.log( "Crazy" );

		return "Hello";
	}

	console.log( foo() );
	// Crazy
	// Hello
	```
	
	finally中的break跳出了当前代码块...

 - switch
	```javascript
	var a = "42";

	switch (true) {
		case a == 10:
			console.log( "10 or '10'" );
			break;
		case a == 42:
			console.log( "42 or '42'" );
			break;
		default:
			// never gets here
	}
	// 42 or '42'
	```
	
	case后面可以跟表达式, 只要跟switch的内容严格相等(===)就行
	
	
	```javascript
	var a = 10;

	switch (a) {
		case 1:
		case 2:
			// never gets here
		default:
			console.log( "default" );
		case 3:
			console.log( "3" );
			break;
		case 4:
			console.log( "4" );
	}
	// default
	// 3
	```
	
	default可以不在最后, 但总是会在最后被判断; 上面代码会首先跳过case1234, 然后匹配到default, 因为default没有break所以会继续往下执行直到case3的break, 这就出现了问题
	
### -script标签
变量提升只会存在于script标签内, 代码错误也只会影响当前script, 不会影响其它script的执行

```javascript
<script>
  var code = "<script>alert( 'Hello World' )</script>";
</script>
```

上面的代码会出错, 在script标签内只要出现"</script>"就会被认为标签结束, 通常的做法是写成"</sc" + "ript>"
