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
  
