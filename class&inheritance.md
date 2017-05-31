定义一个父类:

```
  function Animal (name) {
    this.name = name || 'Animal';
    this.sleep = function(){
      console.log(this.name + '正在睡觉！');
    }
  }
  Animal.prototype.eat = function(food) {
    console.log(this.name + '正在吃：' + food);
  };
```

### Mixin

- 拷贝属性

```
  function mixin( targetObj, sourceObj ) {
    for (var key in sourceObj) {
      targetObj[key] = sourceObj[key];
    }

    return targetObj;
  }

  var Cat = mixin({}, Animal);

  mixin(Cat, {
    name: 'tom'
  });
```
  特点:
  
    1. 实现简单, 可以多继承
    2. 所有属性都需要复制到子实例中, 占用内存
    3. 无法获得父类不可枚举的方法

- 拷贝继承

```
  function Cat(name){
    var animal = new Animal();
    for(var p in animal){
      Cat.prototype[p] = animal[p];
    }
    Cat.prototype.name = name || 'Tom';
  }

  // Test Code
  var cat = new Cat();
  console.log(cat.name);
  console.log(cat.sleep());
  console.log(cat instanceof Animal); // false
  console.log(cat instanceof Cat); // true
```

  特点:

  1. 支持多继承
  2. 效率较低，内存占用高（因为要拷贝父类的属性）
  3. 无法获取父类不可枚举的方法（不可枚举方法，不能使用for in 访问到）
