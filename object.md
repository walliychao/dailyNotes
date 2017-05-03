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

`
  Object.defineProperty( myObject, "a", {
    value: 2,
    writable: true,
    configurable: true,
    enumerable: true
  } );
`

- writable 
  
  值是否可改

- configrable 
  
  descriptor是否可改（不可以重新defineProperty, 不可delete 属性，只可以把writable从true改为false）。

- enumerable 

  是否能在for...in loop中遍历到

  
  
