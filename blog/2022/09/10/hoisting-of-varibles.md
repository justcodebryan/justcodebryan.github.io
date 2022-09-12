---
slug: hoisting-of-varibles
title: Js变量提升问题
authors: [justcodebryan]
tags: [javascript]
---

`JavaScript`中的变量提升问题是一个很常见的面试题目，在阅读《你不知道的 JavaScript》这本书又有了新的感悟。

# 变量提升是什么

首先来看书中的两个例子：

```javascript
// example 1
a = 2;
var a;
console.log(a); // ans：2
```

```javascript
// example 2
console.log(a);
var a = 2; // ans：undefined
```

这两个例子非常简单，解释如下

1. 例 1 中的`var a`变量的声明被提升到了最上面，所以在下面进行赋值的时候，成功地将 2 赋给了`a`这个变量。可以看作是下面这样的代码：

```javascript
var a;
a = 2;
console.log(a);
```

2. 例 2 中可以像上面一样先将变量提升的代码在脑里想一想，得到下面这样的代码：

```javascript
var a;
console.log(a); // 定义了变量a但是没有给它赋值，所以它的值现在是undefined
a = 2;
```

# 函数的变量提升

`js`中的函数同样会进行变量提升这一操作，但是函数有两种声明形式

1. 函数声明 - `function func() { };`
2. 函数表达式 - `var func = function() { };`

需要注意的是函数声明才会将其整个函数提升到整个脚本的最顶上去声明，但是函数表达式会像前面的变量提升一样，先声明变量然后再给它赋值，只不过是赋给变量一个函数(完全可以将其与变量提升一样处理)。在函数表达式的变量提升中，如果出现了先调用然后再声明的情况，会抛出的错误是`TypeError`(类型错误), 而不是`ReferenceError`(引用错误)，这里可以理解为先调用的变量的值应该为`undefined`， 而`undefined`并不是一个函数，不能调用，所以会抛出`TypeError`。

即使是将一个具名的函数赋值给一个变量，也是不能在声明之前调用的：

```javascript
  foo(); // TypeError
  bar(); // ReferenceError

  var foo = function bar() {
    ...
  }
```

经过变量提升以后，代码应该变成以下这种情况了：

```javascript
  var foo;

  foo(); // TypeError
  bar(); // ReferenceError

  foo = function() {
    var bar = ...self...
  }
```

之前在学习变量提升这一块内容的时候，时常会有疑问，如果多个函数变量提升到底会变成什么样？

```
  foo();  // 1

  var foo;

  function foo() {
    console.log(1);
  }

  foo = function() {
    console.log(2);
  }
```

在这段代码中，尽管`var foo`是在`function foo() ...`声明之前的，但是他是重复的声明，所以被忽略了，因此函数声明会被提升到普通变量之前。但是尽管`var`声明会被忽略掉，但是出现在后面的函数声明还是可以覆盖掉前面的

```
  foo();  // 3

  function foo() {
    console.log(1);
  }

  var foo = function() {
    console.log(2);
 }

  function foo() {
    console.log(3);
  }
```

# 总结

- `Javascript`编译器会将`var a = 2`看作`var a`和`a = 2`两个过程
- 函数和变量都会提升
- 声明本身会被提升，但是包括函数表达式的赋值在内的赋值操作不会提升
- 如果函数声明和变量声明同时存在且同名，变量的声明会因为重复被忽略
- 如果多个同名的函数声明，后面的声明会覆盖掉前面的声明
