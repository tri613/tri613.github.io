---
title: '.bind(), .apply() and .call()'
date: 2017-06-15 14:25:31
tags: javascript
---

常常忘記這三個在幹嘛所以整理一下日後好查詢。

<!-- more -->

## .bind()

```
func.bind(thisArg[, arg1[, arg2[, ...]]]);
```

bind doesn't execute function immediately, 
but returns wrapped apply function with certain context (for later execution):

```javascript
function greeting(name) {
    this.hello = this.hello || "Hello";
    return `${this.hello}, ${name}!`;
}

const greetingInJapanese = greeting.bind({hello: "こんばんわ"});

greeting("Trina"); // Hello, Trina!
greetingInJapanese("Trina"); // こんばんわ, Trina!
```

## .apply() / .call()

.call() or .apply() invokes the funciton immediately, and modify the context.

```javascript
func.call(context, argument1, argument2, ...);
func.apply(context, [argument1, argument2, ...]);
```

## Notice

Beware that arrow functions **could not be bound** to another context.

[MDN: Arrow function #No binding of this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions#No_binding_of_this)

## Sources

- http://stackoverflow.com/questions/15455009/javascript-call-apply-vs-bind

- [the `this` bug in javascript](https://pjchender.blogspot.tw/2016/03/javascriptthisbug.html)