---
description: >-
  JavaScript 中没有类的概念的，主要通过原型链来实现继承。继承意味着复制操作，然而 JavaScript
  默认并不会复制对象的属性，相反，JavaScript
  只是在两个对象之间创建一个关联（原型对象指针），这样，一个对象就可以通过委托访问另一个对象的属性和函数，所以与其叫继承，委托的说法反而更准确些。
---

# 原型链及继承

### 原型及原型链

> 当我们 new 了一个新的对象实例，明明什么都没有做，就直接可以访问 toString 、valueOf 等原生方法。那么这些方法是从哪里来的呢？答案就是原型。

![](../.gitbook/assets/__proto__.jpg)

在控制台打印一个空对象时，我们可以看到，有很多方法，已经“初始化”挂载在内置的 `__proto__` 对象上了。这个内置的 `__proto__` 是一个指向原型对象的指针，它会在创建一个新的引用类型对象时（显示或者隐式）自动创建，并挂载到新实例上。这个 `__proto__` 是不可枚举的，当我们尝试访问实例对象上的某一属性 / 方法时，如果实例对象上有该属性 / 方法时，就返回实例属性 / 方法，如果没有，就去 `__proto__` 指向的原型对象上查找对应的属性 / 方法。

### 原型继承的两种常见方式

* 组合继承：子类构造方法中用 call 调用，父类构造方法并传入参数，更改子类构造方法原型为父类实例
  * 优点：1. 实例化时可以传参，不会与父类引用属性共享
  * 缺陷: 因为更改子类构造方法时 new 了父类的实例，子类原型中会存在父类实例属性，浪费内存
* 寄生组合式继承：针对组合继承中，更改子类原型的步骤进行改进，使用 Object.create\(Parent.prototype\) 创建一个新的原型对象赋予子类

    从而解决组合继承的缺陷

组合继承

```javascript
// 组合继承实现

function Parent(value) {
    this.value = value;
}

Parent.prototype.getValue = function() {
    console.log(this.value);
}

function Child(value) {
    Parent.call(this, value)
}

Child.prototype = new Parent();

const child = new Child(1)
child.getValue();
child instanceof Parent;
```

寄生组合集成

```javascript
// 寄生组合继承实现

function Parent(value) {
    this.value = value;
}

Parent.prototype.getValue = function() {
    console.log(this.value);
}

function Child(value) {
    Parent.call(this, value)
}

Child.prototype = Object.create(Parent.prototype, {
    constructor: {
        value: Child,
        enumerable: false,
        writable: true,
        configurable: true
    }
})

const child = new Child(1)
child.getValue();
child instanceof Parent;
```

1. class 继承：class 只是语法糖，本质上还是使用构造函数的原型链继承

```javascript
class Parent {
    constructor(value) {this.value = value;}

    getValue() {console.log(this.value);}
}

class Child extends Parent {
    constructor(value) {super(value)}
}

let child = new Child(1)
child.getValue()
child instanceof Parent
```

