### Object

六种基本类型

- string
- number
- boolean
- null
- undefined
- object

**build-in objects**

- String
- Number
- Boolean
- Object
- Function
- Array
- Date
- RegExp
- Error

除null和undefined，js可以自动将基本类型转换为object。

`object.assign(a, b)` ：shallow copy b 中的普通属性到 a。

**property descriptor**:

`object.getOwnPropertyDescriptor( myObject, "a" )`

类似的方法还有`Object.getOwnPropertyDescriptors(..)`, `Object.getOwnPropertyNames(..)`, `Object.getOwnPropertySymbols(..)`

```javascript
Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
} );
```

- writable  

  值是否可改

- configurable 

  descriptor是否可改（不可以重新defineProperty, 不可delete 属性，只可以把writable从true改为false）。

- enumerable 

  是否能在for...in loop中遍历到
  

**Immutablility**

以下所有的操作都是浅操作，不影响引用指向的其它对象。

- Constant

    writable: false && configurable: false
  
- Object.preventExtensions(..)
    
    防止对象增加新属性
  
- Seal
    
    preventExtension + 现有属性configrable: false 不可新增属性, 且不可配置现有属性

- Freeze
    
    seal + 现有属性writable: false 不可新增属性, 不可配置或更改现有属性值
    
- Object.isExtensible

    检查对象是否可扩展

**getter & setter**

定义`get`或`set`函数后, 属性的value和writable特性会被忽略。

只定义get函数，相当于定义了一个不可写的属性。

**Existence**

检查属性是否存在：`in`会查找prototype链; `hasOwnProperty`只会对象本身的属性。

**Enumerable**

可枚举性相关: 

`propertyIsEnumerable`检查属性是否是对象的直接属性, 且enumerable为true

`Object.keys`返回对象的所有直接的且可枚举的属性的key; `getOwnPropertyNames`只返回对象的直接属性key

遍历:

`for ... in` 会遍历对象自身及prototype上的所有可枚举属性的key

`for ... of` 会遍历数组或有iterator接口的对象的值(跟enumerable无关)

### prototype

 **\___proto___**（隐式原型）vs **prototype**（显式原型）

显式原型: 每一个函数在创建之后都会拥有一个名为prototype的属性，这个属性指向函数的原型对象

隐式原型: JavaScript中任意对象都有一个内置属性[[prototype]]，在ES5之前没有标准的方法访问这个内置属性，但是大多数浏览器都支持通过__proto__来访问。ES5中有了对于这个内置属性标准的Get方法Object.getPrototypeOf()

隐式原型指向创建这个对象的函数(constructor)的prototype

**property in prototype**

修改属性时的查找规则:

1. 如果一个普通value属性foo存在于原型链上, 且非只读(writable: true), 则会在当前对象上创建一个新属性foo并覆盖(shadow)原有属性
2. 如果原型链上存在一个普通value属性, 但是只读(writable: false), 则修改属性操作会失败
3. 如果一个setter属性foo存在于原型链上, 则会调用这个setter属性, 不会发生覆盖操作, 也不会重新定义setter方法
4. 引用属性不会发生覆盖, 只会与原型共用一个属性

在 case2 或 case3 的情况想要覆盖属性只能用`Object.defineProperty`方法

**Object.create**

`Object.create(a)`返回一个新对象, 其原型指向a的原型, `Object.create(null)`会生成一个原型为空的对象

polyfill:

```javascript
if (!Object.create) {
	Object.create = function(o) {
		function F(){}
		F.prototype = o;
		return new F();
	};
}
```

**get & set prototype**

`a.__proto__` <=> `Object.getPrototypeOf( a )` 返回a的原型属性

`Bar.prototype = Object.create( Foo.prototype )` <=> `Object.setPrototypeOf( Bar.prototype, Foo.prototype )`

`setPrototypeOf` 为ES6方法, 设置一个对象的prototype值

**instanceOf & isPrototypeOf**

`a instanceOf Foo` a为待测试对象, Foo 为构造函数 如果Foo.prototype出现在a的整个原型链上, 则返回true

假如Foo是一个bind返回函数, 即 `Foo = bar.bind(this)`, 则Foo.prototype 实际上等于 bar.prototype, a instanceOf Foo 实际上判断的是a是否是bar的实例

`Foo.prototype.isPrototypeOf( a )` 与上面的效果相同, `b.isPrototypeOf(a)` 可以测试两个对象之间的关系

### ES6 Meta Property

**new.target**

在一个constructor函数(由`new`调用的函数)中引用new.target, 返回的是实际new的constructor函数(class中的constructor与class有相同的名字)
```javascript
class Parent {
	constructor() {
		if (new.target === Parent) {
			console.log( "Parent instantiated" );
		}
		else {
			console.log( "A child instantiated" );
		}
	}
}
var a = new Parent();
// Parent instantiated
```

**Symbol.iterator**

可以通过Symbol.iterator获取或设置对象的iterator
```javascript
var it = arr[Symbol.iterator]()
```

**Symbol.toStringTag**

可以通过Symbol.toStringTag修改对象toString时显示的名称
```javascript
function Foo(greeting) {
	this.greeting = greeting;
}

Foo.prototype[Symbol.toStringTag] = "Foo";
var a = new Foo( "hello" );
var b = new Foo( "world" );
b[Symbol.toStringTag] = "cool";
a.toString();				// [object Foo]
String( b );				// [object cool]
```

**Symbol.hasInstance**

可以通过Symbol.hasInstance控制`instanceof`操作符返回的结果
```javascript
Object.defineProperty( Foo, Symbol.hasInstance, {
	value: function(inst) {
		return inst.greeting == "hello";
	}
} );
var a = new Foo( "hello" );
var b = new Foo( "world" );

a instanceof Foo;			// true
b instanceof Foo;			// false
```

**Symbol.species**

Symbol.species是在一个类的原生constructor上的, 可以指定原生方法(**built-in method**)使用的constructor函数, 如`slice`方法创建一个新数组时使用的constructor
```javascript
class Cool {
	// 调用Symbol.species属性返回的是当前Constructor: this
	static get [Symbol.species]() { return this; }

	again() {
		return new this.constructor[Symbol.species]();
	}
}

class Fun extends Cool {}

class Awesome extends Cool {
	// 强制Symbol.species属性返回Cool
	static get [Symbol.species]() { return Cool; }
}

var a = new Fun();
var b = new Awesome();
var c = a.again();
var d = b.again();

c instanceof Fun;		// true
d instanceof Awesome;		// false
d instanceof Cool;		// true
```
上例中Fun的`again`方法使用的是Cool的Symbol.species属性, 返回的是当前使用的constructor, 即`Fun`, 所以`c instanceof Fun`为`true`

而Awesome的Symbol.species属性被改写, 返回Cool的constructor, 因此`d instanceof Cool`会返回`true`

**Symbol.toPrimitive**

Symbol.toPrimitive可以控制`==`或`+`做值比较时由对象转换到原始值时使用的方法`toPrimitive`

```javascript
var arr = [1,2,3,4,5];

arr + 10;				// 1,2,3,4,510

arr[Symbol.toPrimitive] = function(hint) {
	if (hint == "default" || hint == "number") {
		// sum all numbers
		return this.reduce( function(acc,curr){
			return acc + curr;
		}, 0 );
	}
};

arr + 10;				// 25
```
Symbol.toPrimitive会提供一个参数`hint`, 会传入运算类型`string`或`number`或`default`, `default`也表示类型是number

`+`或`==`运算会传入`default`, 表示default类型运算; `*`或`/`会传入`number`, 表示number类型运算; `String(a)`会传入`string`, 表示string类型运算

**Regular Expression Symbol**

ES6新增了一些string的原生方法属性:`Symbol.match`, `Symbol.replace`,`Symbol.search`,`Symbol.split`, 它们都可以传入一个正则表达式对象来进行正则匹配并完成特定字符串处理; 改写这些属性可以改变字符串操作的默认行为, 添加自定义的复杂操作

除非真的有必要, 否则不应该改写这些默认方法, 因为js引擎提供的正则匹配方法通常都是最优化和高效的

**Symbol.isConcatSpreadable**
```javascript
var a = [1,2,3];
var b = [4,5,6];

b[Symbol.isConcatSpreadable] = false;
[].concat( a, b );		// [1,2,3,[4,5,6]]
```
Symbol.isConcatSpreadable属性可以指定一个对象在传给数组的`concat`方法时是否应该作`spread`展开, 可以传`true`或`false`两个值

**Symbol.unscopables**

指定在`with`操作下, 一个对象的哪些属性是不应该暴露在当前作用域下
```javascript
var o = { a:1, b:2, c:3 },
	a = 10, b = 20, c = 30;

o[Symbol.unscopables] = {
	a: false,
	b: true,
	c: false
};

with (o) {
	console.log( a, b, c );		// 1 20 3
}
```
因为给b设了`true`, 所以o对象的b属性不暴露出来, 所以b的值取外层的`20`

因为`with`在`strict`模式下不允许使用, 所以Symbol.unscopables也无效果, 不推荐使用

### ES6 Proxy && Reflect

proxy可以包装任意一个普通的object对象, 拦截对这个对象的一些操作(如get, set等), 做一些特定的操作
```javascript
var obj = { a: 1 };
var handlers = {
	get(target,key,context) {
		// note: target === obj,
		// context === pobj
		console.log( "accessing: ", key );
		return Reflect.get(
			target, key, context
		);
	}
};
var pobj = new Proxy( obj, handlers );

obj.a;
// 1

pobj.a;
// accessing: a
// 1
```
上例中obj是一个普通对象, pobj是一个包装它的proxy对象, 它会拦截obj的所有get操作

参数中target是操作对象, 即obj; key是访问的属性, 这里是a; context是proxy对象, 这里是pobj

proxy创建方式是`new Proxy(target, handler)`, 其中handler是一些统一的处理方法, 这些方法跟Reflect对象上的方法一一对应(名字相同), 配合Proxy handler使用
- get(..): 属性被访问时会被调用处理, 并且在函数中调用Reflect.get(taget, key, context)实际操作目标对象
- set(..): 属性被赋值时
- apply(..): 目标对象是一个方法, 且被调用`call`或`apply`时
- deleteProperty(..): 属性被删除(delete)时
- construct(..): 目标对象作为一个constructor方法被执行(new)时
- getOwnPropertyDescriptor(..): 目标对象(Object)上调用相应方法时
- defineProperty(..): 目标对象(Object)上调用相应方法时
- getPrototypeOf(..): 目标对象(Object)上调用相应方法时
- setPrototypeOf(..): 目标对象(Object)上调用相应方法时
- preventExtensions(..): 目标对象(Object)上调用相应方法时
- isExtensible(..): 目标对象(Object)上调用相应方法时
- ownKeys(..) 目标对象(Object)上调用`Object.keys()`, `Object.getOwnPropertyNames()`, `Object.getOwnPropertySymbols()`, 以及`JSON.stringify(obj)`时, 及获取对象所有属性的时候会调用处理
- enumerate(..): 执行`for...in`操作遍历所有可枚举属性时
- has(..): 目标对象上执行`Object.hasOwnProperty()`或`"prop" in Obj`检查时

**proxy 限制**

以下这些操作都无法被proxy拦截处理
```javascript
typeof obj;
String( obj );
obj + "";
obj == pobj;
obj === pobj;
```

**revokable proxy**

可撤回的proxy: obj, 和handlers同之前的例子, 初始化时调用`Proxy.revocable(obj, handlers)`, 返回的是proxy对象和一个revoke方法

调用revoke方法之后pobj即不生效, 在pobj上进行任何操作都会报TypeError的错误

```javascript
{ proxy: pobj, revoke: prevoke } =
	Proxy.revocable(obj, handlers);

pobj.a;
// accessing: a
// 1

// later:
prevoke();

pobj.a;
// TypeError
```

### ES6 属性顺序

ES6规定了对象的属性在被遍历时: `Reflect.ownKeys()`, `Object.keys()`, `Object.getOwnPropertyNames()`, `Object.getOwnPropertySymbols()`, 以及`JSON.stringify(obj)`时的顺序
- 以升序列出所有整数数字为key的属性(1,2,5,8...)
- 以创建顺序列出所有字符串key
- 以创建顺序列出所有Symbol属性

```javascript
var o = {};

o[Symbol("c")] = "yay";
o[2] = true;
o[1] = true;
o.b = "awesome";
o.a = "cool";

Reflect.ownKeys( o );				// [1,2,"b","a",Symbol(c)]
Object.getOwnPropertyNames( o );	// [1,2,"b","a"]
Object.getOwnPropertySymbols( o );	// [Symbol(c)]
```
