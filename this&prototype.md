### This

`this是运行时而非编译时决定的；即不指向函数本身，也不指向它的作用域，只跟函数执行时的上下文有关。`

#### this绑定规则

- 默认绑定

  ```
    function foo() {
      console.log( this.a );
    }

    var a = 2;

    foo(); // 2
  ```
  这种情况下foo的this指向的是global；在strict模式下this会是undefined。
  
  **需要注意的一个细节是，只有当foo()运行在strict模式时，它的this才会是undefined，foo执行时是否是strict模式不影响。**
  
  ```
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

  ```
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
  
  **注意：**
  
    - 链式调用（obj1.obj2.foo）时，只有直接调用foo的obj2对this有意义。
    
    - 复制函数的引用（ref = obj.foo）时，this并不会被复制，ref的this会根据它执行时的上下文决定。
    
    - 返回callback时，原callback的this同样不会被复制。

        ```
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
      
