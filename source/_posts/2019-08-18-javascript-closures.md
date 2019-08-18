---
layout: _draft
title: JavaScript Closures
tags:
  - JavaScript
  - Closures
comments: false
categories: Coding
date: 2019-08-18 17:17:30
---

> A closure is the capacity of a function to remember/access to the scope of its parent function (a function generator which can accept params, all of then available to the new generated function).

```js
function multiplier(factor) {
  // factor value can be accesed by doPlusFactor after returned
  return function doPlusFactor(number) {
    return number * factor;
  };
}
// multiplier(factor);
var doPlusFactor = multiplier(2);
// the generate a new doPlusFactor has stored the factor value
console.log(`example of closure: ${doPlusFactor(5)}`);
// -> example of closure: 10
```

A good mental model is to think of the `function` keyword as “freezing” the code in its body and wrapping it into a package (the function value/function definition). It's like a var storing _function definition_. Whatever is between the two brackets { } is assigned to the function.

The code inside the function is not evaluated, not executed, just stored into a variable for future use. So when you read `return function(...) {...}`, think of it as returning a handle to a piece of computation, frozen for later use.

In the example, multiplier returns a frozen chunk of code that gets stored in the twice variable. The last line then calls the value in this variable, causing the frozen code `(return number * factor;)` to be activated. It still has access to the `factor` variable from the `multiplier` call that created it, and in addition it gets access to the argument passed when unfreezing it, 5, through its `number` parameter.

## Lexical scope

The idea here is that **we have variables in the local execution context and variables in the global execution context**. How JavaScript looks for variables? If it can’t find a variable in its local execution context, it will look for it in its calling context. And if not found there in its calling context it will repeat that process until reach the global execution context (if it does not find it there, it’s undefined).

```js
let a = 2;
function multiplier(n) {
  return n * a;
}
console.log(`example of scope: ${multiplier(6)}`);
// example of scope: 12
```

In this example the function was able to make de computation after reaching the global context where `a` was defined.

## A Closure or Curried function

Let’s look at an example of a function that returns a function, this is essential to understand closures.
Whenever we declare a new function and assign it to a variable, we store the function definition, as well as a closure.

> **The closure contains all the variables that are in scope at the time of creation of the function**.

Closures let us encapsulate data in a way that it will be available at the time of its execution.

```js
function genGreeter(title) {
  return function(name) {
    return `Hi, ${title} ${name}!`;
  };
}
const titlelizedGreet = genGreeter("Sr");
console.log(titlelizedGreet("Arthur")); // -> Hi, Sr Arthur!
```

When a function returns, its lifecycle is complete. It can no longer perform any work, and its local variables are cleaned up. Unless it returns another function. If that happens, then the returned function still has access to those outer variables, even after the parent passes on. Letting bulid deep augmented scopes.

```js
function genGreeter(salute) {
  return function(title) {
    // salute from parent
    // title from params
    return function(name) {
      // salute from grant parent
      // title from parent
      // name from params
      return `Hi, ${salute} ${title}. ${name}! `;
    };
  };
}
console.log(genGreeter("how are you doing")("Sr")("Arthur")); // -> Hi, Sr Arthur!
```

As the two first functions return new functions they can be invoque in one line `fn()()();`.

## A Closure As privacy encapsulation, private variables

Some say; _JavaScript doesn't support private variables_ but that is not right, one thing is not having private members and another is not being able to acomplish it. It is posible just wrapping a function scope to closure it to the outside word.

```js
const operateAccount = function(initialBalance) {
  // accountBalance is set on fn generation, is not available out of the new fn
  let _accountBalance = initialBalance;
  return {
    getBalance: function() {
      return `Your balance is: $${_accountBalance}`;
    },
    deposit: function(amount) {
      _accountBalance += amount;
    },
    withdraw: function(amount) {
      if (amount > _accountBalance) {
        return "The requested amount is below current balance!";
      }
      _accountBalance -= amount;
      return `Done!, remaining balance: $${_accountBalance}`;
    }
  };
};
const accountManager = operateAccount(0);
accountManager.deposit(1000);
accountManager.withdraw(500);
accountManager.withdraw(300);
console.log(accountManager.getBalance()); // Your balance is: $200
console.log(accountManager.withdraw(300)); // The requested amount is below current balance!
```

Even though the balance can be modified only using the exposed methods can be archived, _operateAccount_ created the `_accountBalance` variable, the three functions it returns all have access to `_accountBalance` via closure.
