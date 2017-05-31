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

```
Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
} );
```

- writable  

  值是否可改

- configrable 

  descriptor是否可改（不可以重新defineProperty, 不可delete 属性，只可以把writable从true改为false）。

- enumerable 

  是否能在for...in loop中遍历到
  

**Immutablility**

以下所有的操作都是浅操作，不影响引用指向的其它对象。

- Constant

    writable: false && configrable: false
  
- Object.preventExtensions(..)
    
    防止对象增加新属性
  
- Seal
    
    preventExtension + 现有属性configrable: false 不可新增属性, 且不可配置现有属性

- Freeze
    
    seal + 现有属性writable: false 不可新增属性, 不可配置或更改现有属性值


**Getters&&Setters**

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

**prototype**

修改属性时的查找规则:

1. 如果一个普通value属性foo存在于原型链上, 且非只读(writable: true), 则会在当前对象上创建一个新属性foo并覆盖(shadow)原有属性
2. 如果原型链上存在一个普通value属性, 但是只读(writable: false), 则修改属性操作会失败
3. 如果一个setter属性foo存在于原型链上, 则会调用这个setter属性, 不会发生覆盖操作, 也不会重新定义setter方法
4. 引用属性不会发生覆盖, 只会与原型共用一个属性

在 case2 或 case3 的情况想要覆盖属性只能用`Object.defineProperty`方法

`Object.getPrototypeOf( a )`返回a.prototype

`Object.create(a)`返回一个新对象, 其原型指向a.prototype

`Bar.prototype = Object.create( Foo.prototype )` <=> `Object.setPrototypeOf( Bar.prototype, Foo.prototype )`

`setPrototypeOf` 为ES6方法设置一个对象的prototype值
