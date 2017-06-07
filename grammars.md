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

ES6的Array.from({length: 3})可以生成一个每个元素都有值(undefined)的数组

### -Number
- |: `a | 0`可以把a转成一个32bit的数字: 去掉小数位, null | NaN | Infinity 转成0

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
- Object.is(): ES6的Object.is方法可以判断一个变量是否是NaN 或 -0
