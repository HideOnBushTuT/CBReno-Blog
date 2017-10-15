---
title: JavaScript 之 this
date: 2017-08-20 23:57:44
tags: JavaScript
---

## 前言

*this* 关键字是 JavaScript 中最复杂的机制之一。之前笔者在使用RN开发的时候碰到了 *this* 的坑，所以看了许多资料，最后总结了这篇文章。侵删！
<!--more-->

## 为什么要使用 this

#### 不用this时，显式传入上下文

```javascript
function identify(context) {
  return context.name.toUpperCase();
}

function speak(context) {
  var greeting = "Hello, I'm " + identify(context);
  console.log(greeting);
}

var me = {
  name: "Kyle"
}

var you = {
  name: "Reader"
}

identify(you); //READER
speak(me); //Hello, I'm KYLE
```

this 提供了优雅的隐式“传递”一个对象引用。可以将API设计的更加简洁易于复用。随着项目的越来越复杂，函数自动引用上下文对象是十分重要的。

## 误解

容易将this理解成指向自身。错误事例：

```javascript
function foo(num) {
  console.log("foo:" + num);
  
  //记录foo的调用次数
  this.count++;
}

foo.count = 0;

var i;
for(i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}

//foo:6
//foo:7
//foo:8
//foo:9

//foo被调用了多少次？
console.log(foo.count); // 0 ???
```



#### 解决方法1

创建另一个带有属性的对象，但是忽视了this的含义和工作原理。

```javascript
function foo(num) {
  console.log("foo:" + num);
  
  //记录foo的调用次数
  data.count++;
}

//创建了带有count属性的对象
var data = {
  count: 0
};

var i;
for(i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}

//foo:6
//foo:7
//foo:8
//foo:9

//foo被调用了多少次？
console.log(data.count); // 4
```

#### 解决方法2

使用函数标识符替代this来引用函数对象

```javascript
function foo(num) {
  console.log("foo:" + num);
  
  //记录foo的调用次数
  //使用函数标识符foo来引用函数对
  foo.count++;
}

foo.count = 0;

var i;
for(i=0; i<10; i++) {
  if (i > 5) {
    foo(i);
  }
}

//foo:6
//foo:7
//foo:8
//foo:9

//foo被调用了多少次？
console.log(foo.count); // 4
```
#### 解决方法3

强制this指向函数对象，通过call方法。

```javascript
function foo(num) {
  console.log("foo:" + num);
  
  //记录foo的调用次数
  this.count++;
}

foo.count = 0;

var i;
for(i=0; i<10; i++) {
  if (i > 5) {
    //使用call(...)可以确保this指向函数对象foo本身
    foo.call(foo, i);
  }
}

//foo:6
//foo:7
//foo:8
//foo:9

//foo被调用了多少次？
console.log(foo.count); // 4
```
## 作用域

this 在任何情况下都不指向函数的词法作用域

作用域“对象”无法通过JavaScript代码访问，存在于JavaScript引擎内部

### this 到底是什么

this在运行时进行绑定，而不是在编写的时绑定，它的上下文取决于函数调用时的各种条件。

当一个函数被调用的时候，会创建一个活动记录（执行上下文），记录包含了函数在哪里被调用（调用栈）、调用方式、传入的参数的信息。this就是这个记录的一个属性，在执行的过程中调用。



## this 全面解析

### 调用位置

* 调用栈，容易出错
* 开发者工具 设置断点或者插入 debugger 调用栈的第二个元素真正的调用位置

### 绑定规则（4种）

#### 默认绑定

默认绑定全局变量，函数直接使用不带有任何修饰的函数引用进行调用 只能使用默认绑定，无法使用其他规则

在**严格模式** （strict mode），则不能将全局变量用于默认绑定， this 会绑定到 **undefined**  

```javascript
function foo() {
  "use strict";
  
  console.log(this.a);
}

var a = 3;

foo(); // TypeError: this is undefined
```



函数运行在非 strict mode 下，才会绑定到全局对象；在严格模式下调用则完全不影响默认绑定

```javascript
function foo() {
  console.log(this.a);
}

var a = 2;

(function() {
  "use strict";
  
  foo(); //2
})();
```


> 通常来说不应该在代码中混合使用 strict 模式和非 strict 模式。在使用第三方库，严格程度和你的代码不同，注意兼容性细节。

#### 隐式绑定

隐式绑定会将函数调用中的this绑定到这个上下文对象中。

```javascript
fucntion foo() {
  console.log(this.a);
}

var obj = {
  a: 2,
  foo: foo
};

obj.foo();  //2
```

对象属性引用链中只有上一层或者最后一层所在的调用位置中起作用。

```javascript
function foo() {
  console.log(this.a);
}

var obj2 = {
  a: 17,
  foo: foo
};

var obj1 = {
  a: 26,
  obj2: obj2
};

obj1.obj2.foo(); //17
```

##### 隐式丢失

隐式丢失就是*被隐式绑定*的函数丢失绑定对象，它会引用默认绑定，从而把this绑定到全局对象或者undefined，取决于是否是严格模式。

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 17,
  foo: foo
};

var bar = obj.foo; //函数别名

var a = "oops, global"; //a是全局对象的属性

bar(); //"oops, global"
```

bar是obj.foo的一个引用，实际上，它引用foo函数本身。所以bar()其实是一个不带任何修饰的函数调用，因此应用了默认绑定。

另一种情况发生在传入函数回调时：

```javascript
function foo() {
  console.log(this.a);
}

function doFoo(fn) {
  //fn 引用的是foo
  
  fn();  //<--调用位置
}

var obj = {
  a: 17,
  foo: foo
};

var a = "oops, global";

doFoo(obj.foo);  //"oops, global"
```

参数传递是一种隐式赋值，传入函数也会被隐式赋值，结果和上面的一样。另外将函数传入语言内置的函数也一样，没有区别。

#### 显示绑定

隐式绑定：必须在一个对象内部包含一个指向函数的属性，通过这个属性间接引用函数，从而把this间接（隐式）绑定到这个对象上。

显示绑定：通过 call(..)和apply(..)方法直接绑定。

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 17
};

foo.call(obj); //2
```

通过foo.call(…)，在调用foo的时候强制将它的this绑定到obj上。

> call(…)和apply(…)的区别在于其他参数上。

但是，显示绑定然无法解决绑定丢失的问题。

##### 硬绑定

可以解决绑定丢失的问题。

```javascript
function foo() {
  console.log(this.a);
}

var obj = {
  a: 17
};

var bar = function() {
  foo.call(obj);
};

bar();  //2
setTimeout(bar, 100); //2

//硬绑定的bar不能修改它的this
bar.call(window); //2
```

* 使用场景一 创建包裹函数，负责接收参数并返回值。

```javascript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}

var obj = {
  a: 17
};

var bar = function() {
  return foo.apply(obj, arguments);
};

var b = bar(3); // 17 3
console.log(b); //20
```

* 使用场景二 创建一个可重复使用的辅助函数。

```javascript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}

//简单的辅助绑定函数
function bind(fn, obj) {
  return function() {
    return fn.apply(obj, arguments);
  };
}

var obj = {
  a: 17
};

var bar = bind(foo, obj);

var b = bar(3); //17 3
console.log(b) //20
```

* 硬绑定常用模式，ES5提供的内置方法 Function.prototype.bind。

```javascript
function foo(something) {
  console.log(this.a, something);
  return this.a + something;
}

var obj = {
  a: 17
};

var bar = foo.bind(obj);

var b = bar(3); // 17 3
console.log(b); //20
```

bind(...)会返回一个硬编码的新函数，它会把你指定的参数设置为this的上下文并调用原始函数。

* API调用的“上下文” 许多第三方库和语言内置函数都提供了可选的参数，作用和bind(..)一样，确保你的回调参数使用指定的this。

```javascript
function foo(el) {
  console.log(el, this.id);
}

var obj = {
  id: "awesome"
};

//调用foo(...)把this绑定到obj
[1, 2, 3].forEach(foo, obj);
//1 awesome 2 awesome 3 awesome
//这些函数都是通过call(..)或者apply(..)实现的显示绑定
```

#### new绑定
> JavaScript中new的机制和面向类的语言完全不同。
>
> 包括内置对象函数在内的所有函数都可以用new来调用，这种函数调用被称为构造函数调用。

使用new来调用函数，会自动执行以下操作：

1. 创建一个全新的对象
2. 新对象被执行[[Prototype]]连接
3. 新对象会绑定到函数调用的this
4. 如果函数没有返回其他对象，new表达式中的函数调用会自动返回这个新对象

```javascript
function foo(a) {
  this.a = a;
}

var bar = new foo(2);
console.log(bar.a); //2
//使用new来调用foo(...)时，会构造一个新的对象并绑定到foo(..)调用中的this。
```

## 优先级

1. 函数是否在 new 中调用（ new 绑定）如果是的话 this  绑定的是新创建的对象。
```javascript
  var bar = new foo()
```
2. 函数是否通过 call、apply（显示绑定）或者硬绑定调用 如果是的话，this 绑定的是指定的对象。

```javascript
  var bar = foo.call(obj2)
```

3. 函数是否在某个上下文对象中调用（隐式绑定）如果是的话，this 绑定的是上下文对象。

```javascript
  var bar = obj1.foo()
```

4. 如果全都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。

```javascript
 var bar = foo()
```


## 例外(ES6箭头函数)
在ES6中有一种无法使用以上四种规则的特殊函数类型：箭头函数。
箭头函数不适用 *this* 的四种标准规则，而是根据外层(函数或者全局)作用域来决定 *this*。

```javascript
function foo() {
    //返回一个箭头函数
  return (a) => {
      //this 继承于foo()
    consolo.log(this.a);
  };
}

var obj1 = {
    a: 2
};

var obj2 = {
    a: 3
};

var bar = foo.call(obj1);
bar.call(obj2);  //输出结果为2， 而不是3.
```

> 箭头函数可以向bind(..)一样确保函数的 *this* 被绑定到指定的对象。另外它更加常见的词法作用域取代了传统的 *this* 机制。在ES6 之前还有一种方法作用和箭头函数一样。

```javascript
function foo() {
  var self = this;  
  setTimeout(function() {
      console.log(self.a);
  }, 100);
}

var obj = {
    a : 2
};

foo.call(obj);  //2
```

`self = this `  和 箭头函数的本质都是想替代 *this* 机制。

## 总结

* 找到函数直接调用的位置，并且使用四种规则来判断 *this* 的绑定对象。
* ES6 中的箭头函数并不会使用四种绑定规则，而是根据当前的词法作用域来决定 *this* 。也就是说箭头函数会继承外层函数调用的 *this* 绑定。和之前的 `self = this` 的作用一致。

## 参考资料
* 《你不是知道的 JavaScript 上卷》