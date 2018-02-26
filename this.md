### This

`this是运行时而非编译时决定的；即不指向函数本身，也不指向它的作用域，只跟函数执行时的上下文有关。`

#### this绑定规则

- 默认绑定

  ```javascript
    function foo() {
      console.log( this.a );
    }

    var a = 2;

    foo(); // 2
  ```
  这种情况下foo的this指向的是global；在strict模式下this会是undefined。
  
  **需要注意的一个细节是，只有当foo()运行在strict模式时，它的this才会是undefined，foo执行时是否是strict模式不影响。**
  
  ```javascript
    function foo() {
      console.log( this.a );
    }

    var a = 2;

    (function(){
      "use strict";

      foo(); // 2
    })();
  ```
  
- 隐式绑定

  ```javascript
    function foo() {
      console.log( this.a );
    }

    var obj = {
      a: 2,
      foo: foo
    };

    obj.foo(); // 2
  ```
  
  这种情况下foo的this指向的是调用它的obj。
  
  注意:
  
  - 链式调用（obj1.obj2.foo）时，只有直接调用foo的obj2对this有意义。

  - 复制函数的引用（ref = obj.foo）时，this并不会被复制，ref的this会根据它执行时的上下文决定。

  - 返回callback时，原callback的this同样不会被复制。

  ```javascript
    function foo() {
      console.log( this.a );
    }

    function doFoo(fn) {
      // `fn` is just another reference to `foo`

      fn(); // <-- call-site!
    }

    var obj = {
      a: 2,
      foo: foo
    };

    var a = "oops, global"; // `a` also property on global object

    doFoo( obj.foo ); // "oops, global"
  ```
      
- 显式绑定

  ```javascript
    function foo() {
      console.log( this.a );
    }

    var obj = {
      a: 2
    };

    foo.call( obj ); // 2
  ```
  
  使用call或apply可以显式的把obj绑定为foo的this。

  如果传入的obj是一个原始值（number, string, boolean）等，会进行装箱操作（boxing），转换为一个对应的对象。

- 强制绑定

  ```javascript
    function foo() {
      console.log( this.a );
    }

    var obj = {
      a: 2
    };

    var bar = function() {
      foo.call( obj );
    };

    bar(); // 2
    setTimeout( bar, 100 ); // 2

    // `bar` hard binds `foo`'s `this` to `obj`
    // so that it cannot be overriden
    bar.call( window ); // 2
  ```
  
  这里强制将foo的this绑定成了obj,不论之后bar函数以什么样的形式执行，都不会改变foo的this绑定。

  ES5提供的bind函数就是一个强制绑定this的方法，下面是一个bind函数的demo：

  ```javascript
    function foo(something) {
      console.log( this.a, something );
      return this.a + something;
    }

    // simple `bind` helper
    function bind(fn, obj) {
      return function() {
        return fn.apply( obj, arguments );
      };
    }

      var obj = {
        a: 2
      };

    var bar = bind( foo, obj );

    var b = bar( 3 ); // 2 3
    console.log( b ); // 5
  ```

  **ES6的bind方法会给新生成的方法的name一个特殊的值，即 bar = foo.bind(obj), bar.name = "bound foo"。**

  **许多新的built-in函数如foreach或js库函数都有thisContext参数，可以显示的指定this值。**

- new绑定

    `在js中，构造函数只是普通的函数碰巧在调用时前面加了个new关键字。因此js中其实没有什么构造函数，只有函数的构造调用。`

    当一个函数作为构造函数被调用时，以下会发生：

    - 一个新的对象会被创建

    - 这个对象会被添加到它的原型链上

    - 这个新创建的对象会被设定为函数构造调用时的的this值

    - 除非这个函数特意返回一个它自己创建的对象，否则函数会默认返回这个新创建的对象作为返回值

#### 执行顺序

`强制绑定 == 显式绑定 > 隐式绑定 > 默认绑定`

new绑定与显式绑定不能同时存在，`new foo.call(obj1)`会报错。new绑定于强制绑定互不干涉（new操作会生成一个新的对象而不影响原来强制绑定函数的this）。

```javascript
  function foo(something) {
    this.a = something;
  }

  var obj1 = {};

  var bar = foo.bind( obj1 );
  bar( 2 );
  console.log( obj1.a ); // 2

  var baz = new bar( 3 );
  console.log( obj1.a ); // 2
  console.log( baz.a ); // 3
```

#### 可能的意外

- foo.call( null ) 
  
  如果null或undefined作为this传给call或apply，会被忽略，foo的this会默认的设为global或window。
  
- (p.foo = o.foo)()

  这句调用可能会认为this是p，其实相当于p.foo = o.foo; foo(); this是默认的global或window。
  
- 箭头函数

  ```javascript
    function foo() {
      // return an arrow function
      return (a) => {
        // `this` here is lexically adopted from `foo()`
        console.log( this.a );
      };
    }

    var obj1 = {
      a: 2
    };

    var obj2 = {
      a: 3
    };

    var bar = foo.call( obj1 );
    bar.call( obj2 ); // 2, not 3!
  ```
  
  箭头函数的this是词法作用域里的this，相当于 var self = this; ((this) => {})(self); 与执行上下文的this不同。
  
  建议词法作用域和执行上下文指定this只使用一种形式，否则会产生混乱。
  
  



