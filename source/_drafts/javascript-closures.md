---
layout: false
title: "javaScript Closures"
comments: false
categories: Coding
tags:
- JavaScript
- Closures
---

A good mental model is to think of the `function` keyword as “freezing” the code in its body and wrapping it into a package (the function value). So when you read `return function(...) {...}`, think of it as returning a handle to a piece of computation, frozen for later use.

```js
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

var twice = multiplier(2);
console.log(twice(5));
// → 10
```

In the example, multiplier returns a frozen chunk of code that gets stored in the twice variable. The last line then calls the value in this variable, causing the frozen code `(return number * factor;)` to be activated. It still has access to the `factor` variable from the `multiplier` call that created it, and in addition it gets access to the argument passed when unfreezing it, 5, through its `number` parameter.
