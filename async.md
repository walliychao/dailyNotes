js是以函数(function)作为最小的代码片段, 并把它们放到事件队列中进行异步执行的. 异步执行的函数可以通过共享公共作用域下的变量来交互数据

#### callback drawbacks

- callback hell / pyramid doom: 嵌套调用难以追踪程序执行顺序
- trust issue: callback通常交给第三方库调用, 必须依赖它在正确的时机调用正确的次数, 并传给正确的参数...

### Promise

- promise一定是异步返回, 即使`Promise(resolve => resolve(42))`的`then()`方法也不会同步调用resolve
- 当一个promise对象resolve的时候, 所有用`then`注册的方法都会按照顺序在`event`队列的当前event之后作为`Jobs`执行; `Jobs`是比`Event`更小的执行单位
- promise不能被撤销, 最终一定会变成`fullfilled`或`rejected`状态, 且状态不可改变; 之后再调用resolve或reject方法都会被忽略
- promise上注册的then方法都会且只会被调用一次
- promise的`resolve`或`reject`方法只接受一个参数, 之后的参数都会被忽略
- promise在创建或`then`方法中如果有报错(`TypeError` `ReferenceError`), 会自动变成`rejected`状态并向之后注册的`reject`方法抛出这个错误

#### promise API

- new Promise(..): 传入一个立即执行的`function(resolve, reject){}`, reject会使当前promise对象变成`rejected`状态; resolve则会根据调用时传入的数据决定, 功能与`Promise.resolve()`相同
- Promise.reject(..) & Promise.resolve(..): `Promise.reject()`会生成一个拒绝状态的promise对象; `promise.resolve()`会根据传入的数据决定, 如果是普通数据就是`fullfilled`状态, 如果是thenable对象则会转化为真正的promise对象, 然后跟进promise对象的最终状态决定是`fullfilled`或`rejected`状态
- then() & catch():
Promise.all([..]) & Promise.race([..]):
