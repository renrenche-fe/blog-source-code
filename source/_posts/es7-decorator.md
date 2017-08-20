---
title: ES7 Decorator
date: 2017-08-20 14:06:59
author: sunshumin
---

decorator是ES7的提案，借鉴于python。
目前浏览器和node都不支持，babel转码器已经支持。

---

## 类的修饰

让我们看一个最简单的类的修饰。

```javascript
function testable(target) {
  target.isTestable = true;
}

@testable
class MyTestableClass {}

console.log(MyTestableClass.isTestable) // true
```

上面的代码中，@testable就是一个修饰器。它修改了MyTestableClass这个类的行为，为它加上了静态属性isTestable。

经过babel转码之后为
```javascript
var _class;

function testable(target) {
    target.isTestable = true;
}

let MyTestableClass = testable(_class = class MyTestableClass {}) || _class;

console.log(MyTestableClass.isTestable); // true
```

可以看出，类的装饰器等同于
```javascript
@decorator
class A {}

// 等同于

class A {}
A = decorator(A) || A;
```

修饰器也可以接收参数，当传参的时候，装饰器函数的第一层的参数为传的参数，第二层的参数为要修饰的类。
```javascript
function testable(isTestable) {
  return function(target) {
    target.isTestable = isTestable;
  }
}

@testable(true)
class MyTestableClass {}
MyTestableClass.isTestable // true

@testable(false)
class MyClass {}
MyClass.isTestable // false
```

babel编码之后为
```javascript
var _dec, _class, _dec2, _class2;

function testable(isTestable) {
    return function (target) {
        target.isTestable = isTestable;
    };
}

let MyTestableClass = (_dec = testable(true), _dec(_class = class MyTestableClass {}) || _class);

MyTestableClass.isTestable; // true

let MyClass = (_dec2 = testable(false), _dec2(_class2 = class MyClass {}) || _class2);

MyClass.isTestable; // false
```


## 方法的修饰

```javascript
class Person {
  @readonly
  name() { return `${this.first} ${this.last}` }
}
```

readonly用来修饰类的name属性，此时的修饰器函数接收三个参数。
```javascript
function readonly(target, name, descriptor){
  // descriptor对象原来的值如下
  // {
  //   value: specifiedFunction,
  //   enumerable: false,
  //   configurable: true,
  //   writable: true
  // };
  descriptor.writable = false;
  return descriptor;
}
```

能看出来`readonly(Person.prototype, 'name', descriptor);`类似于JS的`Object.defineProperty`函数。其实decorator的本质就是利用了ES5的`Object.defineProperty`函数。
📖说明修饰器会修改属性的描述对象，然后被修改的描述对象再去定义属性。

上面的代码经过babel转码之后为
```javascript
var _desc, _value, _class;

function _applyDecoratedDescriptor(target, property, decorators, descriptor, context) {
  var desc = {};
  Object['ke' + 'ys'](descriptor).forEach(function (key) {
    desc[key] = descriptor[key];
  });
  desc.enumerable = !!desc.enumerable;
  desc.configurable = !!desc.configurable;

  if ('value' in desc || desc.initializer) {
    desc.writable = true;
  }

  desc = decorators.slice().reverse().reduce(function (desc, decorator) {
    return decorator(target, property, desc) || desc;
  }, desc);

  if (context && desc.initializer !== void 0) {
    desc.value = desc.initializer ? desc.initializer.call(context) : void 0;
    desc.initializer = undefined;
  }

  if (desc.initializer === void 0) {
    Object['define' + 'Property'](target, property, desc);
    desc = null;
  }

  return desc;
}

let Person = (_class = class Person {
  name() {
    return `${this.first} ${this.last}`;
  }
}, (_applyDecoratedDescriptor(_class.prototype, "name", [readonly], Object.getOwnPropertyDescriptor(_class.prototype, "name"), _class.prototype)), _class);

function readonly(target, name, descriptor) {
  descriptor.writable = false;
  return descriptor;
}
```

举个例子修改属性描述对象的enumerable属性，使得该属性不可遍历。
```javascript
class Person {
  @nonenumerable
  get kidCount() { return this.children.length; }
}

function nonenumerable(target, name, descriptor) {
  descriptor.enumerable = false;
  return descriptor;
}
```

如果同一个方法有多个修饰器时，先从外到内进入，然后由内向外执行。
```javascript
function dec(id){
    console.log('evaluated', id);
    return (target, property, descriptor) => console.log('executed', id);
}

class Example {
    @dec(1)
    @dec(2)
    method(){}
}
// evaluated 1
// evaluated 2
// executed 2
// executed 1
```

以上代码经过babel编码之后为
```javascript
var _dec, _dec2, _desc, _value, _class;

function _applyDecoratedDescriptor(target, property, decorators, descriptor, context) {
    var desc = {};
    Object['ke' + 'ys'](descriptor).forEach(function (key) {
        desc[key] = descriptor[key];
    });
    desc.enumerable = !!desc.enumerable;
    desc.configurable = !!desc.configurable;

    if ('value' in desc || desc.initializer) {
        desc.writable = true;
    }

    desc = decorators.slice().reverse().reduce(function (desc, decorator) {
        return decorator(target, property, desc) || desc;
    }, desc);

    if (context && desc.initializer !== void 0) {
        desc.value = desc.initializer ? desc.initializer.call(context) : void 0;
        desc.initializer = undefined;
    }

    if (desc.initializer === void 0) {
        Object['define' + 'Property'](target, property, desc);
        desc = null;
    }

    return desc;
}

function dec(id) {
    console.log('evaluated', id);
    return (target, property, descriptor) => console.log('executed', id);
}

let Example = (_dec = dec(1), _dec2 = dec(2), (_class = class Example {
    method() {}
}, (_applyDecoratedDescriptor(_class.prototype, 'method', [_dec, _dec2], Object.getOwnPropertyDescriptor(_class.prototype, 'method'), _class.prototype)), _class));
```

## 修饰器不能用于函数

修饰器可以用于类和类的方法，但是不能用于函数，因为存在函数提升。
```javascript
var counter = 0;

var add = function () {
  counter++;
};

@add
function foo() {
}
console.log(counter);
```
上面的代码，意图是执行后counter等于1，但是实际上结果是counter等于0。因为函数提升，使得实际执行的代码是下面这样。

```javascript
@add
function foo() {
}

var counter;
var add;

counter = 0;

add = function () {
  counter++;
};
console.log(counter);
```
由于存在函数提升，使得修饰器不能用于函数。类是不会提升的，所以就没有这方面的问题。

## 实际应用

那在实际应用中，哪里可以用到decorator？

装饰模式经典的应用是`AOP 编程`，比如“日志系统”，日志系统的作用是记录系统的行为操作，它在不影响原有系统的功能的基础上增加记录环节 —— 好比你佩戴了一个智能手环，并不影响你日常的作息起居，但你现在却有了自己每天的行为记录。

更加抽象的理解，可以理解为给数据流做一层filter，因此 AOP 的典型应用包括 安全检查、缓存、调试、持久化等等。


- 实现简单的输出日志

```javascript
class Math {
  @log
  add(a, b) {
    return a + b;
  }
}

function log(target, name, descriptor) {
  var oldValue = descriptor.value;

  descriptor.value = function() {
    console.log(`Calling "${name}" with`, arguments);
    return oldValue.apply(null, arguments);
  };

  return descriptor;
}

const math = new Math();

// passed parameters should get logged now
math.add(2, 4);
// Calling "add" with Arguments(2) [2, 4]
```

- 牛人实现的常用的decorator，[jayphelps/core-decorators](https://github.com/jayphelps/core-decorators)。

@lazyInitialize
防止属性被初始化，直到被装饰的属性被实际查找。

使用
```javascript
class Editor {
  @lazyInitialize
  hugeBuffer = createHugeBuffer();
}
```

怎么利用decorator实现的呢？
```javascript
import { decorate, createDefaultSetter } from './private/utils';
const { defineProperty } = Object;

function handleDescriptor(target, key, descriptor) {
  const { configurable, enumerable, initializer, value } = descriptor;
  return {
    configurable,
    enumerable,

    get() {
      // This happens if someone accesses the
      // property directly on the prototype
      if (this === target) {
        return;
      }

      const ret = initializer ? initializer.call(this) : value;

      defineProperty(this, key, {
        configurable,
        enumerable,
        writable: true,
        value: ret
      });

      return ret;
    },

    set: createDefaultSetter(key)
  };
}

export default function lazyInitialize(...args) {
  return decorate(handleDescriptor, args);
}
```

- 利用 decorator 实现 AOP

```javascript
function aop(before, after) {
    return function(target, key, descriptor) {
        const method = descriptor.value;
        descriptor.value = (...args) => {
            let ret;

            before && before(...args);
            ret = method.apply(target, args);
            if (after) {
                ret = after(ret);
            }

            return ret;
        };
    };
}

function beforeTest1(opt) {
    opt.name = opt.name + ', haha';
}

function beforeTest2(...args) {}

function afterTest2(ret) {
    console.log('haha, add 10!');
    return ret + 10;
}

class Test {
    @aop(beforeTest1)
    static test1(opt) {
        console.log(`hello, ${opt.name}`);
    }

    @aop(beforeTest2, afterTest2)
    static test2(...args) {
        return args.reduce((a, b) => a + b);
    }
}
```

任何装饰者模式的代码都可以通过 decorator 来实现。
还有一些例子，例如[GoogleChrome/decorators-es7](https://github.com/GoogleChrome/samples/tree/gh-pages/decorators-es7/read-write)，不再叙述。


## 装饰者模式

在不改变对象自身的基础上，在程序运行期间给对像动态的添加职责。优点是把类（函数）的核心职责和装饰功能区分开，实现对象的动态扩展能力。

```javascript
var Plan1 = {
    fire: function () {
        console.log('发射普通的子弹');
    }
};

var missileDecorator= function () {
    console.log('发射导弹!');
};

var fire = Plan1.fire;

Plan1.fire=function () {
    fire();
    missileDecorator();
};

Plan1.fire();
```

