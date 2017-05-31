
定义一个父类:

```
function Animal (name) {
	this.name = name || 'Animal';
  	this.sleep = function() {
    		console.log(this.name + '正在睡觉！');
  	}
}
Animal.prototype.eat = function(food) {
  	console.log(this.name + '正在吃：' + food);
};
```

### call&apply 实现继承

```
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

```
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

### 构造函数继承

```
function Cat(name){
	Animal.call(this);
	this.name = name || 'Tom';
	var sleep = this.sleep;
	this.sleep = function() {
		console.log(this.name + 'is going to sleep');
		sleep();
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

```
function Cat(name){
	var instance = new Animal();
	instance.name = name || 'Tom';
	var sleep = instance.sleep;
	instance.sleep = function() {
		console.log(this.name + '去窝里');
		sleep.call(this);
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

```
function Cat(name){
	var animal = new Animal();
	for(var p in animal){
		Cat.prototype[p] = animal[p];
	}
	Cat.prototype.name = name || 'Tom';
	this.sleep = function() {
		console.log(this.name + '去窝里');
		animal.sleep.call(this);
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

```
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

```
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
