---
description: javascript 中是没有类的概念的，通过原型链实现继承
---

# 继承及原型链



## 原型继承的两种常见方式

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

