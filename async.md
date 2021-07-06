js是以函数(function)作为最小的代码片段, 并把它们放到事件队列中进行异步执行的. 异步执行的函数可以通过共享公共作用域下的变量来交互数据

### Callback

```javascript
ajax('post', function(success, err) {})
```
```javascript
setTimeout(function() {}, 1000)
```

#### callback drawbacks

- callback hell / pyramid doom: 嵌套调用难以追踪程序执行顺序
- trust issue: callback通常交给第三方库调用, 必须依赖它在正确的时机调用正确的次数, 并传给正确的参数...

### Promise

```javascript
var p = new Promise( function(resolve,reject){
	resolve( "A" );
} );
p.then(function(a) {
	console.log(a);
	return "B";
})
.then(function(b) {
	console.log(b);
});
```

- promise一定是异步返回, 即使`Promise(resolve => resolve(42))`的`then()`方法也不会同步调用resolve
- 当一个promise对象resolve的时候, 所有用`then`注册的方法都会按照顺序在`event`队列的当前event之后作为`Jobs`执行; `Jobs`是比`Event`更小的执行单位
- promise不能被撤销, 最终一定会变成`fullfilled`或`rejected`状态, 且状态不可改变; 之后再调用resolve或reject方法都会被忽略
- promise上注册的then方法都会且只会被调用一次
- promise的`resolve`或`reject`方法只接受一个参数, 之后的参数都会被忽略
- promise在创建或`then`方法中如果有报错(`TypeError` `ReferenceError`), 会自动变成`rejected`状态并向之后注册的`reject`方法抛出这个错误
- promise的`then`方法会返回一个promise对象, 如果当前then方法内有抛错则会变成`rejected`状态, 否则变成`fullfilled`状态, resolve方法如有返回值则会传到下个resolve方法中作为参数

#### promise API

- new Promise(..): 传入一个立即执行的`function(resolve, reject){}`, reject会使当前promise对象变成`rejected`状态; resolve则会根据调用时传入的数据决定, 功能与`Promise.resolve()`相同
- Promise.reject(..) & Promise.resolve(..): `Promise.reject()`会生成一个拒绝状态的promise对象; `promise.resolve()`会根据传入的数据决定, 如果是普通数据就是`fullfilled`状态, 如果是thenable对象则会转化为真正的promise对象, 然后跟进promise对象的最终状态决定是`fullfilled`或`rejected`状态
- then() & catch(): 每个promise对象都可以用`then`注册`fullfilled`或`rejected`状态时的处理方法; `catch`相当于then(null, ..); then或catch方法都会返回一个新的promise对象方便链式调用. 如果不传参数或参数不是一个方法时会有默认的处理方法, fullfill回调默认把数据传给下一个then注册的回调, reject回调也会把错误传给下一个
- Promise.all([..]) & Promise.race([..]): 数组中需要传promise对象或thenable对象, 最终会转化成合法的promise对象, `promise.all`在所有成员都`fullfilled`时才会变成`fullfilled`状态, 任何一个成员`rejected`就会变成`rejected`状态; `promise.race`相反, 任何一个成员`fullfilled`即变成`fullfilled`状态, 只有所有成员都`rejected`的时候才会变成`rejected`状态

```javascript
promise.then(func1, func2)
promise.then(func1).catch(func2)
```
区别: 第一行代码`func1`与`func2`只有一个会执行, 如果`func1`执行出错则没有地方处理; 第二行中不论`promise`还是`func1`抛错都有兜底函数`func2`处理

#### promise 代码实现
```javascript
var Promise = function(callback) {
    this.successFns = [];
    this.errorFns = [];
    this.state = 'pending';
    var resolve = res => {   // resolve和reject需要保留外层this值
        if (res instanceof Promise) {
            return res.then(resolve, reject);
        }
        // 保证resolve和reject异步调用, 实际应该使用MutationObserver实现微任务
        setTimeout(() => {  
            this.state = 'fulfilled';
            this.value = res;
            this.successFns.forEach(fn => fn(res));
        })
    }
    var reject = err => {
        setTimeout(() => {
            this.state = 'rejected';
            this.value = err;
            this.errorFns.forEach(fn => fn(err));
        })
    }
    try {
        callback(resolve, reject);
    } catch (e) {
        reject(e);
    }
};
Promise.prototype.then = function(successFn, errorFn) {
    var self = this;
    successFn = typeof successFn === 'function' ? successFn : data => data;
    errorFn = typeof errorFn === 'function' ? errorFn : data => data;
    return new Promise(function(resolve, reject) {
        switch(self.state) {
            case 'fulfilled':
                return new Promise((resolve, reject) =>
                    resolve(successFn(self.value)));
                break;
            case 'rejected':
                return new Promise((resolve, reject) =>
                    resolve(errorFn(self.value)));
                break;
            default:
                return new Promise((resolve, reject) => {
                    self.successFns.push(function(res) {
                        resolve(successFn(res));
                    });
                    self.errorFns.push(function(res) {
                        resolve(errorFn(res));
                    });
                });
                
        }
    })
}
Promise.prototype.catch = function(errorFn) {
    return this.then(null, errorFn)
};
Promise.prototype.resolve = function(value) {
    return new Promise((resolve, reject) => {
        resolve(value);
    })
}
Promise.prototype.reject = function(value) {
    return new Promise((resolve, reject) => {
        reject(value);
    })
}
Promise.prototype.all = function(promiseArray) {
    return new Promise((resolve, reject) => {
        try {
            let resultArray = Array.from({length});
            const length = promiseArray.length;
            let received = 0;
            for (let i = 0;;i++) {
                promiseArray[i].then(data => {
                    resultArray[i] = data;
                    received++;
                    if (received === length) {
                        resolve(resultArray);
                    }
                }, reject);
            }
        } catch(e) {
            reject(e);
        }
    })
}
Promise.prototype.race = function(promiseArray) {
    return new Promise((resolve, reject) => {
        try {
            for (let i = 0;;i++) {
                promiseArray[i].then(data => 
                    resolve(data), err => 
                    reject(err)
                );
            }
        } catch(e) {
            reject(e);
        }
    })
}
```

#### promisify & thunkify

```javascript
function promisify(fn) {
	return function() {
		var args = [].slice.call( arguments );
		return new Promise( function(resolve,reject){
			fn.apply(
				null,
				args.concat( function(err,v){
					if (err) {
						reject( err );
					}
					else {
						resolve( v );
					}
				} )   // currify
			);
		} );
	};
}
```
```javascript
function thunkify(fn) {
	return function() {
		var args = [].slice.call( arguments );
		return function(cb) {
			args.push( cb );
			return fn.apply( null, args );  // currify
		};
	};
}
```

- promisify返回一个函数, 这个函数可以把传入的callback形式代码转化为相应的promise对象
- thunkify返回一个函数, 这个函数把传入的方法柯里化, 转化成只需要最后的callback参数的形式
- promisify也可以实现柯里化, 把调用时传入的参数固化下来, 作为之后函数调用时参数的一部分

使用形式:
```javascript
// symmetrical: constructing the question asker
var fooThunkory = thunkify( foo );
var fooPromisory = promisify( foo );

// symmetrical: asking the question
var fooThunk = fooThunkory( 3, 4 );
var fooPromise = fooPromisory( 3, 4 );

// get the thunk answer
fooThunk( function(err,sum){
	if (err) {
		console.error( err );
	}
	else {
		console.log( sum );		// 7
	}
} );

// get the promise answer
fooPromise
.then(
	function(sum){
		console.log( sum );		// 7
	},
	function(err){
		console.error( err );
	}
);
```
co的实现就是基于thunkify, 之后作者在转向promise实现

#### promise + reduce

按顺序执行异步函数并打印结果
```javascript
let array = Array.from({length: 5}, () => {
      return (Math.random() * 10000) | 0
})
const textPromiseFuncs = array.map((v, i) => {
    return function() {
        return new Promise((resolve, reject) => {
            setTimeout(() => resolve(i + '  ' + v), v)
        })
    }
});
// log them in order
textPromiseFuncs.reduce((chain, textPromiseFunc) => {
    return chain.then(textPromiseFunc)
    .then(text => console.log(text));
}, Promise.resolve());
```
结果按序号依次打印:
```
0  1484
1  4070
2  6546
3  8000
4  9251
```
上面的代码是串行执行, 如果需要并行执行则直接创建promise数组
```javascript
const textPromises = array.map((v, i) => {
    return new Promise((resolve, reject) => {
	setTimeout(() => resolve(i + '  ' + v), v)
    })
});
// log them in order
textPromises.reduce((chain, textPromise) => {
    return chain.then(() => textPromise)
    .then(text => console.log(text));
}, Promise.resolve());
```
这样promise在数组map执行时即并行开始执行, 使用`Promise.all`也可以实现并行执行异步请求但顺序打印
```javascript
Promise.all(textPromises).then(resolveArray => console.log(resolveArray.join('\n')))
```
> 与使用reduce的区别在于`Promise.all`需要等到所有promise都resolve之后一次性返回结果数组,
> 而reduce在promise1返回时即打印结果, promise2返回时假设promise3已经返回, 则promise2和promise3都会打印
> 因此用reduce是效率最高的方式

### Generator

```javascript
function *foo(x) {
	var y = x * (yield "Hello");	// <-- yield a value!
	return y;
}
var it = foo( 6 );
var res = it.next();	// first `next()`, don't pass anything
res.value;				// "Hello"
res = it.next( 7 );		// pass `7` to waiting `yield`
res.value;				// 42
```

- `it = foo(6)`返回一个iterator, 但不实际执行foo方法
- `it.next()`开始执行foo方法, 到`yield "hello"`暂停, 并将"Hello"作为iterator的value返回
- `it.next(7)`从yield开始执行foo方法, 并将7传给yield语句(代替`yield "hello"`执行计算)
- 最后y作为iterator的value返回(如果没有return则会默认返回undefined)

generator把原先原子化执行的函数(function)转化成了代码执行片段, 且可以利用yield和next动态跟外部互相传递数据

#### yield delegation

```javascript
function *foo() {
	console.log( "`*foo()` starting" );
	yield 3;
	yield 4;
	console.log( "`*foo()` finished" );
}

function *bar() {
	yield 1;
	yield 2;
	yield *foo();	// `yield`-delegation!
	yield 5;
}

var it = bar();

it.next().value;	// 1
it.next().value;	// 2
it.next().value;	// `*foo()` starting
					// 3
it.next().value;	// 4
it.next().value;	// `*foo()` finished
					// 5
```

- 可以以`yield *foo()`的形式在yield之后接另一个generator方法, 执行到这一句之后会转到foo中执行
- 被delegate的foo方法可以return数据给外层的yield语句, 错误或异常也可以`throw`给外部`catch`
- `yield *[ "B", "C", "D" ]`也可以实现delegate, 因为数组有默认的iterator(只要返回合法的iterator, 即使不是generator函数也能是现在yield delegate)

### callback + generator

```javascript
function foo(x,y) {
	ajax(
		"http://some.url.1/?x=" + x + "&y=" + y,
		function(err,data){
			if (err) {
				// throw an error into `*main()`
				it.throw( err );
			}
			else {
				// resume `*main()` with received `data`
				it.next( data );
			}
		}
	);
}
```
```javascript
function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
var it = main();
it.next();
```

- 在`yield foo(11, 31)`时调用异步的foo方法, 跟进ajax返回结果判断继续执行main方法还是抛出错误
- 可以用`it.throw()`向generator外部抛出错误, 如果`yield foo(11, 31)`时发生错误, 也可以从外部catch错误
- `it.return()`可以提前结束iterator遍历

### promise + generator

```javascript
function foo(x,y) {
	return request(
		"http://some.url.1/?x=" + x + "&y=" + y
	);
}
function *main() {
	try {
		var text = yield foo( 11, 31 );
		console.log( text );
	}
	catch (err) {
		console.error( err );
	}
}
```
```javascript
var it = main();
var p = it.next().value;
// wait for the `p` promise to resolve
p.then(
	function(text){
		it.next( text );
	},
	function(err){
		it.throw( err );
	}
);
```

- `request`会返回一个promise, 即yield一个promise对象, 然后在promise的注册方法里控制generator的iterator执行
- generator的`main`函数内部可以不作任何变化, 只是外部的处理由回调变成了promise

### promise + generator + runner

```javascript
function run(gen) {
	var args = [].slice.call( arguments, 1), it;

	// initialize the generator in the current context
	it = gen.apply( this, args );

	// return a promise for the generator completing
	return Promise.resolve()
		.then( function handleNext(value){
			// run to the next yielded value
			var next = it.next( value );

			return (function handleResult(next){
				// generator has completed running?
				if (next.done) {
					return next.value;
				}
				// otherwise keep going
				else {
					return Promise.resolve( next.value )
						.then(
							// resume the async loop on
							// success, sending the resolved
							// value back into the generator
							handleNext,

							// if `value` is a rejected
							// promise, propagate error back
							// into the generator for its own
							// error handling
							function handleErr(err) {
								return Promise.resolve(
									it.throw( err )
								)
								.then( handleResult );
							}
						);
				}
			})(next);
		} );
}
```
```javascript
function *main() {
	// ..
}

run( main );
```

- `run`方法返回一个promise对象, 而不需要自己手动执行`it.next().value`
- promise对象`fullfilled`的时候, 如果generator函数未结束, 会自动把返回值转化成promise对象, 在该promise`fullfilled`时执行generator的下一步操作
- generator每一个yield的值, 都会转化成promise对象, 在它`fullfilled`的时候异步的执行generator的下一步, 如果generator结束则直接返回value

### async + await

```javascript
async function request(url) {
	var resp = await (
		new Promise( function(resolve,reject){
			var xhr = new XMLHttpRequest();
			xhr.open( "GET", url );
			xhr.onreadystatechange = function(){
				if (xhr.readyState == 4) {
					if (xhr.status == 200) {
						resolve( xhr );
					}
					else {
						reject( xhr.statusText );
					}
				}
			};
			xhr.send();
		} )
	);

	return resp.responseText;
}

var pr = request( "http://some.url.1" );

pr.then(
	function fulfilled(responseText){
		// ajax success
	},
	function rejected(reason){
		// Oops, something went wrong
	}
);
```

- `request()`返回一个promise对象, promise`fullfilled`时会自动执行generator下一步, 即`resolve(xhr)`执行时xhr会传到`var resp = await ...`处, 而不是main的then方法里
- request方法`return resp.responseText`会传到main返回的promise对象的then方法中作为fullfilled方法的参数
- async方法会在promise `resolve`时自动执行下一步代码直到结束, 并且返回值会传给async返回的promise对象的then方法中, 相当于 promise + generator + runner的语法糖

async方法一定会返回一个promise, 这个promise会以函数的返回值resolve, 会以函数的抛错reject
