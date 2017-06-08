### -Function
```
function a(b, c) {}
a.length    // 2
```
函数对象的length属性表示它定义时参数的个数

### -typeof
```
typeof null === 'object'
typeof NaN === 'number'
```

```
var a
typeof a === 'undefined'
typeof b === 'undefined'
```
未声明和未赋值的变量typeof都是undefined

### -hoist
```
if (0) {
    var a = 3;
}
a; // undefined
```

```
if (0) {
    a = 3;
}
a; // referenceError
```

如果用var定义, 即使未执行if内的语句, 也会进行变量提升

### -undefined
`undefined`是一个内置的变量, 默认值就是undefined

在non-strict模式下可以改变全局的undefined值, strict模式下不可以
```
function foo() {
	undefined = 2; // really bad idea!
}
```

在strict和non-strict模式下都可以改变本地的undefined值
```
function foo() {
	"use strict";
	var undefined = 2;
	console.log( undefined ); // 2
}
```

**绝对不要改undefined值!!**

`void`可以把任何表达式的值变成undefined 返回
```
void 0      // undefined
void 3+4    // undefined
```

### -Array
```
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
	```
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

### 类型转换
一个对象转换成特定的原始值类型时, 会首先检查是否有`valueOf`方法, 有则调用这个方法; 否则检查是否有`toString`方法将对象转化为原始值之后再做类型转换; 如果两个方法都没有, 则会报`TypeError`

- Date to Number
	```
	var d = new Date( "Mon, 18 Aug 2014 08:53:06 CDT" );
	+d; // 1408369986000
	```
- Symbol
	```
	var s1 = Symbol( "cool" );
	String( s1 );					// "Symbol(cool)"
	s` + "";						// TypeError
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
```
parseInt( 0.000008 );		// 0   ("0" from "0.000008")
parseInt( 0.0000008 );		// 8   ("8" from "8e-7")
parseInt( false, 16 );		// 250 ("fa" from "false")
parseInt( parseInt, 16 );	// 15  ("f" from "function..")

parseInt( "0x10" );		// 16
parseInt( "103", 2 );		// 2
```

### -双等号==
- ==两边的变量都是object(function, array)时, 只有当变量指向的对象完全一致时才会返回true, 而且跟===的表现一样, 不会做类型转换
	```
	var aa = [1,2,3]
	var bb = [1,2,3]
	aa == bb              // false
	aa === bb             // false

	var aa = [1,2,3]
	var bb = aa 
	aa == bb              // true
	aa === bb             // true
	```
