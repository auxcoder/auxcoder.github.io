---
title: AngularJS Promises
date: 2017-03-20 10:36:21
categories: Coding
tags:
  - AngularJS
  - Promises
  - $q
author: Auxcoder
---

## AngularJS use the `$q` library, a [service](https://docs.angularjs.org/api/ng/service/$q) to build promises.

A promise is always in either one of three states:

- _Pending_ - the result has not been computed yet
- _Fulfilled_ - the result was computed successfully
- _Rejected_ - a failure occurred during computation

```js
function myPromise(){
	// A deferred object can be created within an asynchronous function
	var deferred = $q.defer();
	// do an action
	// if action is satisfied
	if (){
		// resolved promise
		deferred.resolve(reason); // Fulfilled
	}
	else {
		// reject promise
		deferred.rejected(reason); // Rejected
	}
	// The function should return a promise object
	return deferred.promise // default Pending
}
```

The first call `deferred.resolve(...)` puts the promise into the fulfilled state. As result a success callback will be invoked. The second call `deferred.reject(...)` puts the promise into the rejected state. As result an error callback will be invoked. It is also possible to invoke `deferred.notify(...)` to track chaining.

We can use promises directly in controllers to chain call that depends one in other.

```js
function myAsyncFunction(myParams) {
  var deferred = $q.defer();
  deferred.notify({ action: "get-items", message: "Getting items ..." });

  // update/notify should be called after returning they deferred object
  $timeout(function() {
    deferred.notify("getItems was called");
  }, 0);

  // MyService.getItems() / Service.method()
  MyService.getItems(myParams).then(
    // success callback
    function(response) {
      deferred.resolve(response);
    },
    // success callback
    function(response) {
      deferred.reject(response);
    }
  );
  return deferred.promise;
}
```

We can write the similar functionality in shorter way returning the `$q` object.

```js
function myAsyncFunction(myParams) {
  return $q(function(resolve, reject) {
    MyService.getItems(resolve, reject);
  });
}
```

We can use the promise chain `.then(successCallback, [errorCallback], [notifyCallback])` to wait until promise resolve then do something with the data. The `.finally(callback, notifyCallback)` allows you to observe either the fulfillment or rejection of a promise, but to do so without modifying the final value.

```js
myAsyncFunction({ param: "some data" })
  .then(
    // on success
    function(response) {
      console.log(response);
    },
    // on error
    function(response) {
      console.log(response);
    },
    // on notify
    function(response) {
      console.log(response); // 'getItems was called'
    }
  )
  .finally(function(response) {
    console.log(response);
  });
```

> `finally` will called no matter if the promise was resolved or rejected

## Chain promises with AngularJS

Let say that we have three service method and the second depend on first and third in second, resulting in a chain like: `first() -> second(firstResponse) -> third(secondResponse)`.

Let say the we have a service like this.

```js
angular.module("myApp", []).factory("myService", function($q) {
  return {
    first: function(params) {
      return $q(function(resolve, reject) {
        $http
          .get("http://myendpoint.com/segment", { params: params })
          .then(resolve, reject);
      });
    },
    second: function(params) {
      return $q(function(resolve, reject) {
        $http
          .get("http://myendpoint.com/segment", { params: params })
          .then(resolve, reject);
      });
    },
    third: function(params) {
      return $q(function(resolve, reject) {
        $http
          .get("http://myendpoint.com/segment", { params: params })
          .then(resolve, reject);
      });
    }
  };
});
```

And in our controller we call the one of the available methods in the service and `then` call another and so on.

```js
myService
  .callFirst()
  .then(
    // on success
    function(firstResult) {
      return myService.second({ param: firstResult.someAttr });
    }
  )
  .then(
    // on success
    function(secondResult) {
      return myService.third({ param: secondResult.someAttr });
    }
  )
  .then(
    // on success
    function(thirdResult) {
      console.log(thirdResult);
    }
  );
```

It turns out that we can easily extend the promise continuing the chain by tacking on a second then. Down the chain, each then invocation returns a new derived promise that becomes part of the chain. _Each derived promise is resolved with the return value of each then function_.

```js
myService
  .callFirst()
  .then(
    // on success
    function(firstResult) {
      // the returned object is available in the next .then()
      return firstResult.length;
    }
  )
  .then(function(count) {
    console.log(count);
  });
```

The promise resolve value/object is available as input to the next then method. This is true even if the then method doesn't explicitly return something. In that case the

```js
myService
  .callFirst()
  .then(
    // on success
    function(firstResult) {
      // the returned object is available in the next .then()
      console.log(firstResult.length);
    }
  )
  .then(function(count) {
    console.log(count); // undefined as nothing was explicitly returned from the previous promise
  });
```

## Handling errors

Error handling can be added through catch methods. You can either add a granular catch handler to deal with specific promises or you can add a single catch all at the end. The main difference is that specific error handlers will handle the error and let subsequent then handlers carry on as if nothing happened.

```js
function handleError() {
  console.log(response);
}
function handleError() {
  console.log(response);
}

myService
  .callFirst()
  .then(
    // on success
    function(firstResult) {
      // the returned object is available in the next .then()
      console.log(firstResult.length);
    },
    handleError,
    handleUpdate
  )
  .then(
    function(count) {
      console.log(count); // undefined as nothing was explicitly returned from the previous promise
    },
    handleError,
    handleUpdate
  );
```

The catch function may seem like a different beast, but it's really just a then method. `catch(errorCallback)` is shorthand for `promise.then(null, errorCallback)`.

```js
myService
  .callFirst()
  .then(
    // on success
    function(response) {
      console.log(response);
    }
  )
  .catch(function(e) {
    console.log(e);
  });
```

We have a `finally(callback, notifyCallback)` method that allows you to observe either the fulfillment or rejection of a promise.

```js
myService
  .callFirst()
  .then(
    // on success
    function(response) {
      console.log(response);
    }
  )
  .catch(
    // on error
    function(e) {
      console.log(e);
    }
  )
  .finally(function() {
    console.log("promise ends with success or error");
  });
```

## `$q.all()`

```js
function myAsyncFunction() {
  var deferred = $q.defer();

  var houses = MyService.getHouses();
  var cars = MyService.getCars();

  $q.all([houses, cars]).then(
    // success callback
    function(response) {
      deferred.resolve(response);
    },
    // success callback
    function(response) {
      deferred.reject(response);
    }
  );
  return deferred.promise;
}
```

## `$q.reject(reason)`

Some times we need to reject a promise chain base on some data processing or specific validation.
Let's assume that we have the functions `callFirst` and `callSecond` that returns a promise.

```js
callFirst({
  attr_a: "a",
  attr_b: "b"
})
  .then(
    // on success
    handle_callFirst
  )
  .then(handle_callSecond)
  .catch(
    // on error
    handle_rejected
  )
  .finally(function() {
    console.log("promise ends with success or error");
  });

function handle_callFirst(response) {
  var _brand = response[0];
  return callSecond({ attr_c: _brand.name });
}

function handle_callSecond() {
  var _myResultsFromSecond = response;
}

function handle_rejected(e) {
  console.log(e);
}
```

As you can see in the `handle_callFirst` we return a new promise to chain it. That function call needs a value from the previous response. Suppose that we need to filter out the response to use the brand that has products on it `_brand.has_products === true`, in this case if any brand in the collection has _has_products_ equal true then second call is going to fail as `_brand` will be `null`;
In this type of case we can reject the promise right there to throw the rejection as the second call needs that data.

```js
function handle_callFirst(response) {
  var _brand = response.filter(function(x) {
    return x.has_products === true;
  });
  if (_brand.length === 0) return $q.reject(reason);
  return callSecond({ attr_c: _brand.name });
}
```

### Use examples

Reads a local file and returns it’s content

```js
function readFile(fileBlob) {
  var deferred = $q.defer();
  var reader = new FileReader();
  reader.onload = function() {
    deferred.resolve(reader.result);
  };
  reader.onerror = function() {
    deferred.reject();
  };

  try {
    reader.readAsDataURL(fileBlob);
  } catch (e) {
    deferred.reject(e);
  }

  return deferred.promise;
}
```

## What about native browser Promises and Async/Await

### Async

Let’s start with the `async` keyword.

```js
async function f() {
  return 1;
}
```

Then wwe can use `then()`.

```js
async function isPromise() {
  return 1;
}
isPromise()
  .then(alert)
  .catch();
```

The modifier `async` ensures that the function returns a promise, and wraps non-promises in it. There’s another keyword, `await`, that works only inside async functions.

Let’s start with the `async` keyword.

### Await

It works only inside async functions.

```js
async function isPromise() {
  let promise = new Promise((resolve, reject) => {
    setTimeout(() => resolve("done!"), 1000);
  });

  // wait till the promise resolves or rejects
  return await promise;
}

isPromise();
```

<!---
*References*
+ [Promises in AngularJS. Part II. $q service](https://www.webcodegeeks.com/javascript/angular-js/promises-angularjs-part-ii-q-service/)
+ [Angular promise chaining explained](http://www.syntaxsuccess.com/viewarticle/angular-promise-chaining-explained)
+ [AngularJS Promises - The Definitive Guide](http://www.dwmkerr.com/promises-in-angularjs-the-definitive-guide/)
+ [JavaScript Promises and AngularJS $q Service](http://www.webdeveasy.com/javascript-promises-and-angularjs-q-service/)
+ [Async/await](https://javascript.info/async-await)
+ [AngularJS Promises ($q)](https://thinkster.io/a-better-way-to-learn-angularjs/promises)
--->
