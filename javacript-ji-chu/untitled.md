---
description: javascript 基础部分
---

# javascript 基础

1. 七个基本类型：String Number Boolean BigInt undefined null Symbol
2. 引用类型：Function Array Object
3. typeof 除了 null 是 object 能够识别其他基本类型
4. instanceof 原理是通过原型链来判断类型，因此只能判断引用类型，对基本类型值无效
5. this 的绑定优先级：new -&gt; bind/apply -&gt; obj.func\(\) -&gt; func\(\)
6. bind 只有在第一次的时候绑定的 this 有效
7. 箭头函数没有自己的 this 需要依赖父作用域中的 this
8. 深拷贝方法：
   * JSON.parse\(JSON.stringify\(\)\)
   * MessageChannel
   * 自己写一个
9. JSON.parse\(JSON.stringify\(\)\)深拷贝的问题：
   * 会忽略 undefined、function、Symbol
   * 对象内部有循环引用会报错
10. MessageChannel 深拷贝问题：可以正确复制 undefined 以及循环引用，但是函数和 Symbol 属性还是会报错
11. MessageChannel 实现的深拷贝：

```javascript
function structuralClone(obj) {
    return new Promise((resolve, reject) => {
        const {port1, port2} = new MessageChannel();
        port1.onmessage = ev => resolve(ev.data);
        port2.postMessage(obj);
    })
}

var obj = {
    a: 1,
    c: {
        d: undefined,
        e: 'helo world'
    }
}

const test = async () => {
  const clone = await structuralClone(obj)
  console.log(clone)
}
test()
```

12. 手写一个简单的深拷贝

```javascript
function deepClone (obj) {
    function isObject (obj) {
        const type = typeof obj;
        return (type === 'object' && obj != null) || type === 'function'
    }

    if (!isObject) throw new Error('非对象')

    const isArray = Array.isArray(obj);
    const isFunction = typeof obj === 'function';
    let newObj = isArray ? [...obj] : isFunction ? obj : {...obj}
    Reflect.ownKeys(newObj).forEach(key => {
        newObj[key] = isObject(newObj[key]) ? deepClone(newObj[key]) : newObj[key]
    })

    return newObj
}
```

