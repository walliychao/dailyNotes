js是以函数(function)作为最小的代码片段, 并把它们放到事件队列中进行异步执行的. 异步执行的函数可以通过共享公共作用域下的变量来交互数据

#### callback drawbacks
- callback hell / pyramid doom: 嵌套调用难以追踪程序执行顺序
- trust issue: callback通常交给第三方库调用, 必须依赖它在正确的时机调用正确的次数, 并传给正确的参数...

### Promise
promise resolve或reject一个异步的值(不会立即返回), 且一旦返回这个值就是不可变的(immutable); 不能cancel一个promise
