
定义一个父类:

```javascript
function Animal (name) {
	this.name = name || 'animal';
  	this.sleep = function() {
    		console.log(this.name + ' is sleeping');
  	}
}
Animal.prototype.eat = function(food) {
  	console.log(this.name + ' is eating ' + food);
};
```

### call&apply 实现继承

```javascript
var animal = new Animal();
var cat = {
	sleep: function() {
		animal.sleep.call( this );
	}
};

cat.sleep();
```

利用call/apply将其它对象的方法变成自己的方法

### prototype原型继承

```javascript
function Cat(){ 
}
Cat.prototype = new Animal();
Cat.prototype.name = 'cat';

//　Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); //true 
console.log(cat instanceof Cat); //true
```

特点:

  1. 非常纯粹的继承关系，实例是子类的实例，也是父类的实例
  2. 父类新增原型方法/原型属性，子类都能访问到
  3. 要想为子类新增属性和方法，必须要在new Animal()这样的语句之后执行，不能放到构造器中
  4. 无法实现多继承
  5. 来自原型对象的引用属性是所有实例共享的
  6. 创建子类实例时，无法向父类构造函数传参
  
### 原型继承变体

```javascript
function Cat(){ 
}
Cat.prototype = Object.create(Animal.prototype);
Cat.prototype.name = 'cat';

//　Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); //true 
console.log(cat instanceof Cat); //true
```

特点:与原型继承类似; 只继承原型方法和属性, 不继承实例方法; 不执行构造函数, 避免了构造函数执行时的side-affect

### 构造函数继承

```javascript
function Cat(name){
	Animal.call(this);
	this.name = name || 'Tom';
	var sleep = this.sleep;
	this.sleep = function() {
		sleep();
		console.log(this.name + ' is purring');
	};
}

// Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); // false
console.log(cat instanceof Cat); // true
```

特点:

  1. 解决了原型链继承中子类实例共享父类引用属性的问题
  2. 创建子类实例时，可以向父类传递参数
  3. 可以实现多继承（call多个父类对象）
  4. 实例并不是父类的实例，只是子类的实例
  5. **只能继承父类的实例属性和方法，不能继承原型属性/方法**
  6. 无法实现函数复用，每个子类都有父类实例函数的副本，影响性能

### 构造继承变体

- 实例继承/寄生继承

```javascript
function Cat(name){
	var instance = new Animal();
	instance.name = name || 'Tom';
	var sleep = instance.sleep;
	instance.sleep = function() {
		sleep.call(this);
		console.log(this.name + ' is purring');
	};
	return instance;
}

// Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); // false
```

特点:

  1. 不限制调用方式，不管是new 子类()还是子类(),返回的对象具有相同的效果
  2. 实例是父类的实例，不是子类的实例
  3. 不支持多继承

- 拷贝继承

```javascript
function Cat(name){
	var animal = new Animal();
	for(var p in animal){
		Cat.prototype[p] = animal[p];
	}
	Cat.prototype.name = name || 'Tom';
	this.sleep = function() {
		animal.sleep.call(this);
		console.log(this.name + ' is purring');
	}
}

// Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); // false
console.log(cat instanceof Cat); // true
```

特点:

  1. 手动复制父类方法给子类, 支持多继承
  2. 效率较低，内存占用高（因为要拷贝父类的属性）
  3. 无法获取父类不可枚举的方法（不可枚举方法，不能使用for in 访问到）
    
### 构造原型继承组合

- 组合继承

```javascript
function Cat(name){
	Animal.call(this);
	this.name = name || 'Tom';
}
Cat.prototype = new Animal();

// Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); // true
```

特点:
  
  1. 弥补了构造继承的缺陷，可以继承实例属性/方法，也可以继承原型属性/方法
  2. 既是子类的实例，也是父类的实例
  3. 不存在引用属性共享问题
  4. 父类构造函数可传参
  5. 函数可复用
  6. 调用了两次父类构造函数，生成了两份实例（子类实例将子类原型上的那份屏蔽了）
  
- 寄生组合继承

```javascript
function Cat(name){
	Animal.call(this);
	this.name = name || 'Tom';
}
(function(){
	// 创建一个没有实例方法的类
	var Super = function(){};
	Super.prototype = Animal.prototype;
	//将实例作为子类的原型
	Cat.prototype = new Super();
})();

// Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); //true
```

特点: 砍掉了父类的实例属性, 拥有组合继承的所有优点, 且不会初始化两次实例方法, 缺点是实现复杂

- ES5写法

```javascript
function Cat(name){
	Animal.call(this);
	this.name = name || 'Tom';
}
Cat.prototype = Object.create(Animal.prototype);
// 继承Animal上的静态方法和属性
Object.setPrototypeOf(Cat, Animal);

// Test Code
var cat = new Cat();
console.log(cat.sleep());
console.log(cat instanceof Animal); // true
console.log(cat instanceof Cat); //true
```

特点: 拥有寄生组合式的所有优点, 且写法更简单

### OOLO(Objects Linked to Other Objects)继承

父对象:
```javascript
var Animal = {
	name: 'animal',
	sleep: function() {
		console.log(this.name + ' is sleeping');
	},
	eat: function(food) {
		console.log(this.name + ' is eating' + food);
	}
}
```

子对象:
```javascript
var Cat = {
	init: function(name) { this.name = name },
	catSleep: function() {
		this.sleep();
		console.log(this.name + ' is purring');
	}
}
```

父子对象继承:
```javascript
Object.setPrototypeOf(Cat, Animal)
```

新增子级对象:
```javascript
cat1 = Object.create(Cat);
cat1.init('cat');
```

验证对象间关系:
```javascript
Animal.isPrototypeOf(cat1);
Cat.isPrototypeOf(cat2);
Object.getPrototypeOf(cat1) === Cat;
```

**特点**:

1. 没有构造函数或类和实例的概念, 没有`new`、 `instanceOf`、`constructor`, 只是把对象链接到原型链上, 实现更简单更接近原理
2. 不能通过`new + param`直接初始化一个对象, 必须先Object.create再init, setup等
3. 使用ES5, ES6新语法, 需要浏览器支持或代码编译

### ES6 Class继承

父类:
```javascript
class Animal {
	constructor(name) {
		this.name = name || 'animal';
	}
	sleep() {
		console.log(this.name + ' is sleeping');
	}
	eat(food) {
		console.log(this.name + ' is eating ' + food);
	}
}
```

子类:
```javascript
class Cat extends Animal {
	constructor(name) {
		super(name);
	}
	sleep() {
		super.sleep();
		console.log(this.name + ' is purring ');
	}
}
```

实例化子类:
```javascript
let cat1 = new Cat('cat1');
let cat2 = new Cat('cat2');
```

**特点**:
1. 是原型链继承的语法糖, 只是语法更简洁优雅, 更像类定义的格式
2. super会调用原型链上上一层的对象的构造函数, 类似组合继承的模式; super是类定义时静态确定的, 后期修改对象的原型, super不会改变
3. 不要把class继承模式跟其它方式混用, 不要定义之后动态修改类的原型对象, 否则会出问题
