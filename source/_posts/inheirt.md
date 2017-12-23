---
title: js原生继承
date: 2015-07-03
top: 200
img: image/inheirt.jpg
categories: [web前端,基础]
tags: [继承]
---
  js中的继承有多种实现方式，今天我们来分析一下各种继承的方式以及其优缺点。
 <!--more-->   
### 原型链继承  
   首先介绍一下原型链的基本概念。    
   先来理解一下原型、构造函数和实例之间的关系。      
   * 每个构造函数都有一个原型对象（通过prototype属性）    
   * 原型对象都包含一个指向构造函数的指针（通过constructor属性）
   * 实例都包含一个指向原型对象的内部指针（通过隐式\_\_proto\_\_属性）
   
   
   那么，若原型对象被另一个原型的实例覆盖，则此时原型对象将包含一个指向另一个原型对象的指针。按照这样层层递进，就构成了实例与原型的链条。
   
   这个概念可能不好理解，请看下面例子：
 ```javascript
  /**
   * 父类
   * @param {any} name
   */
  function Parent(name) {
      this.name = name;
      this.age = 10;
  }
  Parent.prototype.sayName = function() {
      console.log('parent name is ' + this.name + '!');
  };
  /**
   * 子类
   * @param {any} name
   */
  function Child(name) {
      this.name = name;
  }
  // 使用原型链继承Parent构造函数
  Child.prototype = new Parent('father');
  // constructor属性，指向当前的Child构造函数。
  Child.prototype.constructor = Child;
  Child.prototype.sayName = function() {
      console.log('child name is ' + this.name + '!');
  };
  // 调用父类
  var parent = new Parent('father');
  parent.sayName(); // parent name is father!
  // 调用子类
  var child = new Child('son');
  child.sayName(); // child name is son!
  child.age = 18;
  console.log(child.age); // 18
  ```
  例子中通过将Child的原型对象覆盖，将Child与Parent关联起来了。
 
  使用原型链继承缺点：
  * 由于包含引用类型值的原型属性会被所有实例共享，导致修改一个实例引用值，所有的实例对于这个引用全部被修改。请看下面例子：
  
  
  ```javascript
  function Parent() {
      this.colors = ["red","blue","green"];
      this.name = "super";
  }
  function Child() {
  }
  Child.prototype = new Parent();
  var instance1 = new Child();
  instance1.colors.push("black");
  instance1.name = "sub";
  console.log(instance1.name); // sub
  console.log(instance1.colors); //red, blue, green, black
  var instance2 = new Child();
  console.log(instance2.name); // super
  console.log(instance2.colors); //red, blue, green, black
  ```
  就像上面的例子所示，当我们new一个新对象时（其过程可查看new一个新对象会发生什么），原型属性会复制一份到我们实例中。对于值类型，实例会复制其名字和值放在另一块内存中；而对于引用类型，实例只是复制了指向它的值的指针。因而修改实例的值类型，不会影响其他实例；但是修改引用类型的值，其他实例也会被影响到。
  
* 创建子类型的实例时，并不能在不影响所有对象实例的情况下给超类型的构造函数传递参数。 

* 顺序不当可能会引发问题，如Child.prototype.sayName 必须写在 Child.prototype = new Parent(‘father’); 之后，不然就会被覆盖掉。
  
### 构造函数继承
  基本思想：在子类型构造函数内部调用超类型构造函数。
  还是先看例子：
  ```javascript
  /**
   * 父类
   * @param {any} name
   */
  function Parent(name) {
      this.name = name;
      this.age = 10;
  }
  Parent.prototype.sayName = function() {
      console.log('parent name is ' + this.name + '!');
  };
  Parent.prototype.doSomthing = function() {
      console.log('parent do something!');
  };
  /**
   * 子类
   * @param {any} name
   */
  function Child(name) {
      // 通过call,apply 改变对象的this指向来继承
      Parent.call(this, name);
      // Parent.apply(this, arguments);
  }
  Child.prototype.sayName = function() {
      console.log('child name is ' + this.name + '!');
  };
  var child = new Child('son');
  child.sayName(); // child name: son
  child.doSomthing(); // TypeError: child.doSomthing is not a function
  ```
  
 相当于 Parent 函数在 Child 函数中执行了一遍，并且将所有与 this 绑定的变量都切换到了 Child 上，使用构造函数继承的方式解决了原型链继承的问题：
   * 实例可以独享一份引用类型的值
     通过call改变this指向，这样每次执行Parent函数，this指向的都是新的对象。相当于每个新的对象都有一份完整的Parent代码。即每个实例都有一份自己的属性副本。
  
* 可以传参数
我们可以通过call函数向Parent传参数   
   
构造函数继承也有缺点：
  * 无法实现函数复用
  看上面的代码可知，我们只是执行了Parent函数，但是并没有继承它的原型链上的函数。这样会导致若要使用公有函数时，自己定义或者在Parent构造函数中定义，违背了函数复用的初衷。
  
  
### 组合继承

组合继承将原型链继承和构造函数继承结合到一起，是最常用的的继承方式。
基本思想是：使用原型链实现对原型属性和方法的继承，借用构造函数实现对实例属性的继承。  
```javascript
/**
 * 父类
 * @param {any} name
 */
function Parent(name) {
    this.name = name;
    this.age = 10;
}
Parent.prototype.sayName = function() {
    console.log('parent name is ' + this.name + '!');
};
Parent.prototype.doSomthing = function() {
    console.log('parent do something!');
};
/**
 * 子类
 * @param {any} name
 */
function Child(name) {
    // 通过call,apply 改变对象的this指向来继承
    Parent.call(this, name);  // 第二次调用 
    // Parent.apply(this, arguments);
}
Child.prototype = new Parent('father');  // 第一次调用
Child.prototype.constructor = Child;
Child.prototype.sayName = function() {
    console.log('child name is ' + this.name + '!');
};
var child = new Child('son');
child.sayName(); // child name: son
child.doSomthing(); // parent do something!
```

由例子可知，组合继承结合了原型链继承和构造函数继承的优点，既可以拥有属于自己的属性，也有了共同的方法。
当然，它也有缺点。

* 无论什么情况下，都会调用两次超类型构造函数。一次在创建子类型的原型时；一次在子类型构造函数内部。当第二次调用时，会重写第一次调用时获得的原型属性。

### 原型式继承
基本思想：借助已有的对象创建新对象，不必通过构造函数。

```javascript
var ob = { name: '李达康', friends: ['沙瑞金', '季昌明'] };
/**
 * 原型式继承  参数o,引用类型值，实质就是一个内存地址
 * @param {any} obj
 * @returns
 */
function object(obj) {
    /**
     * 创建一个构造函数F
     */
    function F() {
        // 空构造函数F
    }
    F.prototype = obj;
    return new F();
}
var ob1 = object(ob);
ob1.name = '侯亮明';
ob1.friends.push('陈海');
console.log(ob1.name); // 侯亮明
console.log(ob1.friends); // ["沙瑞金", "季昌明", "陈海"]
var ob2 = object(ob);
console.log(ob2.name); // 李达康
console.log(ob2.friends); // ["沙瑞金", "季昌明", "陈海"]
```
可以看出该方法与原型链继承类似，但是写法比它简单。
因此，若只是想让一个对象和另一个相似，可以使用这种方法。
不过它也存在缺点：
* 包含引用类型值的属性始终共享相应的值。

### 寄生式继承
基本思想：创建一个仅用于封装继承过程的函数，在内部对对象做相关增强，然后返回。

```javascript
var ob = { name: '李达康', friends: ['沙瑞金', '季昌明'] };
// 上面再ECMAScript5 有了一新的规范写法，Object.create(ob) 效果是一样的
/**
 * 创建一个对象
 * @param {any} o
 * @returns
 */
function createOb(o) {
    var newob = Object.create(o); // 创建对象
    newob.sayname = function() { // 增强对象
        console.log(this.name);
    };
    return newob; // 指定对象
}
var ob1 = createOb(ob);
ob1.sayname(); // 李达康
```

这样做相当于构造函数那样，并不是真正的函数复用。而且包含引用类型值的属性依然始终共享相应的值。

### 寄生组合式继承

这个方法属于比较完美的方法。先看代码：

```javascript
/**
 * 实现继承
 * @param {any} Parent 父类
 * @param {any} Child  子类
 */
function inheritPrototype(Parent, Child) {
    Child.prototype = Object.create(Parent.prototype); // 修改
    Child.prototype.construtor = Child;
}
/**
 * 父类
 * @param {any} name
 */
function Parent(name) {
    this.name = name;
    this.friends = ['达康', '瑞金'];
}
Parent.prototype.sayName = function() {
    console.log('parent name is ' + this.name + '!');
};
/**
 * 子类
 * @param {any} name
 * @param {any} parentName
 */
function Child(name, parentName) {
    Parent.call(this, parentName);
    this.name = name;
}
// 实现继承
inheritPrototype(Parent, Child);
Child.prototype.sayName = function() {
    console.log('child name is ' + this.name + '!');
};
var parent = new Parent('father');
parent.sayName(); // parent name: father
var child1 = new Child('son', 'father');
child1.friends.push('猴子'); // ["达康", "瑞金", "猴子"]
console.log(child1.friends);
child1.sayName(); // child name: son
var child2 = new Child('son2', 'father');
console.log(child2.friends); // ["达康", "瑞金"]
child1.sayName(); // child name: son2
```

这个方法解决了组合继承调用两次超类型的缺点。

首先回顾一下组合继承的两次调用：
* 创建子类型的原型对象时调用（new Parent()）
这个过程会主要是new一个对象的过程，它会复制Parent的属性和方法给子类型。

* 在子类型构造函数里调用
在构造函数里调用时，又会复制一遍超类型的属性，因而会影响性能。

对于寄生组合式继承方式：
* 先将超类型中的原型对象复制一份，再new对象作为子类型的原型对象。这样做，我们只是复制了超类型的原型对象，而对于超类型构造函数里的属性不会复制。因而减少了调用超类型的次数。
* 这样做仍然保持原型链不变

### ES6中class实现继承
* ES6提供了更接近传统语言”类”的写法，引入了Class（类）这个概念，作为对象的模板。通过class关键字，可以定义类。

* 基本上，ES6的class可以看作只是一个语法糖，它的绝大部分功能，ES5都可以做到；新的class写法只是让对象原型的写法更加清晰、更像面向对象编程的语法而已。

```javascript
/**
 * 父类
 * @class Parent
 */
class Parent {
    /**
     * Creates an instance of Parent.
     * @param {any} name
     * @memberOf Parent
     */
    constructor(name) {
        this.name = name;
    }
    /**
     * 做点东西
     * @memberOf Parent
     */
    doSomething() {
        console.log('parent do something!');
    }
    /**
     * 打印名字
     * @memberOf Parent
     */
    sayName() {
        console.log(`parent name is ${this.name}!`);
    }
}
/**
 * 子类 继承 父类
 * @class Child
 * @extends {Parent}
 */
class Child extends Parent {
    /**
     * Creates an instance of Child.
     * @param {any} name
     * @param {any} parentName
     * @memberOf Child
     */
    constructor(name, parentName) {
        // 调用基类的构造方法
        super(parentName);
        this.name = name;
    }
    /**
     * 打印名字 覆盖父类的sayName方法
     * @memberOf Child
     */
    sayName() {
        console.log(`child name is ${this.name}!`);
    }
}
const child = new Child('son', 'father');
child.sayName(); // child name: son
child.doSomething(); // parent do something!
const parent = new Parent('father');
parent.sayName(); // parent name: father
```

如果项目中使用到ES6语法开发，推荐使用ES6继承
