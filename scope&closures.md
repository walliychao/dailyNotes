### 编译原理

- 词法分析 Tokenizing
- 语法分析 生成语法分析树（AST）
- 代码生成

`JS是解释型语言，代码编译之后马上执行。`

### 编译器、执行引擎、作用域的关系

编译器解析代码，生成作用域。引擎查找作用域，并根据作用域执行代码。

`编译器声明变量和函数，引擎查找并执行赋值操作、垃圾回收。`

### LHS&RHS
- LHS: left hand search, 赋值操作符左侧查找，即引用查找
- RHS: right hand search, 复制操作符右侧查找，即值查找

`js引擎跟作用域的交互基本可以归结为值查找和引用查找。`

### 词法作用域

作用域在函数及变量声明时就已经确定。

只有两种方法：eval, with可以改变词法作用域，但会导致**效率变差**。

- eval会在执行时动态改变代码的词法作用域（在eval内有声明语句）。strict模式下则只做用在eval内部，外部引用会出现引用错误。

- with(obj)会用obj创建一个新的作用域。使用时只能修改obj已有的属性值，修改不存在的属性时会泄露到global作用域中（strict模式不允许with）。

### 作用域生成

- 函数作用域

- 命名空间
  将某个对象或函数：global, object作为命名空间
  
#### 匿名函数的优缺点

优点：不污染外部命名空间，节省代码量。

缺点：

- 不易调试，在 call stack 里看不到函数名。
- 函数没法引用自身，arguments.callee已被淘汰。
- 影响代码可读性。

``js只有函数作用域。``

例外（块作用域）：

- with(obj): with后面会生成一个新的块作用域。

- try/catch: catch(err)后面会生成一个新的块作用域，在catch外部err引用非法。

- let/const: es6的变量声明都是块作用域内部有效的。

### 变量提升

``变量、函数声明会提升到（函数）作用域的顶部。``

只有函数声明会提升，函数表达式不会。且函数声明比变量声明更高。

重复的变量声明，之后的会被忽略；重复的函数声明会覆盖前面的声明。

### 闭包

  闭包就是一个函数能够在它的词法作用域之外记忆并且访问它的词法作用域内容。
  
  ``function return fn: 将函数引用传到作用域外``
  
  #### 循环+闭包
  
  - 添加匿名函数作为闭包
  ``
    for (var i=1; i<=5; i++) {
      (function(){
        var j = i;
        setTimeout( function timer(){
          console.log( j );
        }, j*1000 );
      })();
    }
  ``
  ``
    for (var i=1; i<=5; i++) {
      (function(j){
        setTimeout( function timer(){
          console.log( j );
        }, j*1000 );
      })( i );
    }
  ``
  
  - 使用**let**, 创造block作用域
  
  ``
    for (var i=1; i<=5; i++) {
      let j = i; // yay, block-scope for closure!
      setTimeout( function timer(){
        console.log( j );
      }, j*1000 );
    }
``
``
  for (let i=1; i<=5; i++) {
    setTimeout( function timer(){
      console.log( i );
    }, i*1000 );
  }
``

#### js Module
