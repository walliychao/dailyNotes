### Function Object
```
function a(b, c) {}
a.length    // 2
```
函数对象的length属性表示它定义时参数的个数

### typeof
```
typeof null === 'object'
```

```
var a
typeof a === 'undefined'
typeof b === 'undefined'
```
未声明和未赋值的变量typeof都是undefined

### hoist
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
