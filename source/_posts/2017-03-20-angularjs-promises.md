---
title: AngularJS Promises
date: 2017-03-20 10:36:21
categories: Coding
tags:
- AngularSJS
- Promises
- $q
author: auxcoder
---

## AngularJS use the `$q` library,a [service](https://docs.angularjs.org/api/ng/service/$q) to build promises.

```js
// A deferred object can be created within an asynchronous function
var deferred = $q.defer();

// The function should return a promise object
return deferred.promise
```

A promise is always in either one of three states:

+ *Pending* - the result hasn’t been computed yet
+ *Fulfilled* - the result was computed successfully
+ *Rejected* - a failure occurred during computation

The first call `deferred.resolve(...)` puts the promise into the fulfilled state. As result a success callback will be invoked. The second call `deferred.reject(...)` puts the promise into the rejected state. As result an error callback will be invoked. It is also possible to invoke `deferred.notify(...)` to track chaining.

We can use promises directly in controllers to chain call that depends one in other.

```js
function myAsyncFunction(myParams){
	var deferred = $q.defer();
	deferred.notify({action: 'get-items' , message: 'Getting items ...'});

	// update/notify should be called after returning ther deferred object
	$timeout(function(){
		deferred.notify('getItems was called');
	}, 0);

	// MyService.getItems() / Service.method()
	MyService.getItems(myParams).then(
		// success callback
		function(response){
			deferred.resolve(response);
		},
		// success callback
		function(response){
			deferred.reject(response);
		}
	);
	return deferred.promise;
}
```

We can write the similar functionality in shorter way returning the `$q` object.

```js
function myAsyncFunction(myParams){
	return $q(function (resolve, reject) {
		MyService.getItems(resolve, reject);
	});
}
```

We can use the promise chain `.then()` to wait untill promise resolve then do something with the data.

```js
myAsyncFunction({param: 'some data'})
.then(
	// on success
	function(response){
		console.log(response);
	},
	// on error
	function(response){
		console.log(response);
	},
	// on notify
	function(response){
		console.log(response); // 'getItems was called'
	},
)
.finally(
	function(response){
		console.log(response);
	}
);
```
> `finally` will called no matter if the promise was resolved or rejected

## Chain promises with AngularJS
Let say that we have three service method and the second depend on first and third in second, resulting in a chain like: `first() -> second(firstResponse) -> third(secondResponse)`.

Let say the we have a service like this.

```js
angular.module('myApp', []).factory('myService', function($q) {
	return {
		first: function (params) {
			return $q(function (resolve, reject) {
				$http.get('http://myendpoint.com/segment', {params: params}).then(resolve, reject);
			});
		},
		second: function (params) {
			return $q(function (resolve, reject) {
				$http.get('http://myendpoint.com/segment', {params: params}).then(resolve, reject);
			});
		},
		third: function (params) {
			return $q(function (resolve, reject) {
				$http.get('http://myendpoint.com/segment', {params: params}).then(resolve, reject);
			});
		}
	}
});
```

And in our controller we call the one of the available methods in the service and `then` call another and so on.

```js
myService.callFirst()
	.then(
		// on success
		function(firstResult){
			return myService.second({param: firstResult.someAttr});
		}
	)
	.then(
		// on success
		function(secondResult){
			return myService.third({param: secondResult.someAttr});
		}
	)
	.then(
		// on success
		function(thirdResult){
			console.log(thirdResult);
		}
	);
```

It turns out that we can easily extend the promise continuing the chain by tacking on a second then. Down the chain, each then invocation returns a new derived promise that becomes part of the chain. *Each derived promise is resolved with the return value of each then function*.

```js
myService.callFirst()
	.then(
		// on success
		function(firstResult){
			// the returned object is available in the next .then()
			return firstResult.lenght;
		}
	)
	.then(function(count){
		console.log(count);
	})
```

The promise resolve value/object is available as input to the next then method. This is true even if the then method doesn't explicitly return something. In that case the

```js
myService.callFirst()
	.then(
		// on success
		function(firstResult){
			// the returned object is available in the next .then()
			console.log(firstResult.lenght);
		}
	)
	.then(function(count){
		console.log(count); // undefined as nothing was explicity returned from the previous promise
	})
```

## Handling errors

Error handling can be added through catch methods. You can either add a granular catch handler to deal with specific promises or you can add a single catch all at the end. The main difference is that specific error handlers will handle the error and let subsequent then handlers carry on as if nothing happened.

```js
function handleError(){
	console.log(response)
}
function handleError(){
	console.log(response)
}

myService.callFirst()
	.then(
		// on success
		function(firstResult){
			// the returned object is available in the next .then()
			console.log(firstResult.lenght);
		},
		handleError,
		handleUpdate
	)
	.then(
		function(count){
			console.log(count); // undefined as nothing was explicity returned from the previous promise
		},
		handleError,
		handleUpdate
	);
```

The catch function may seem like a different beast, but it's really just a then method. `catch(errorCallback)` is shorthand for `promise.then(null, errorCallback)`.

```js
myService.callFirst()
	.then(
		// on success
		function(response){
			console.log(response);
		}
	)
	.catch(
		function(e){
			console.log(e);
		}
	);
```

We have a `finally(callback, notifyCallback)` method that allows you to observe either the fulfillment or rejection of a promise.

```js
myService.callFirst()
	.then(
		// on success
		function(response){
			console.log(response);
		}
	)
	.catch(
		function(e){
			console.log(e);
		}
	).finally(
		function(){
			console.log('promise ends with success or error');
		}
	);
```

## `$q.all()`

```js
function myAsyncFunction(){
	var deferred = $q.defer();

	var houses = MyService.getHouses();
	var cars = MyService.getCars();

	$q.all([houses, cars]).then(
		// success callback
		function(response){
			deferred.resolve(response);
		},
		// success callback
		function(response){
			deferred.reject(response);
		}
	);
	return deferred.promise;
}
```

<!---
*References*
+ [Promises in AngularJS. Part II. $q service](https://www.webcodegeeks.com/javascript/angular-js/promises-angularjs-part-ii-q-service/)
+ [Angular promise chaining explained](http://www.syntaxsuccess.com/viewarticle/angular-promise-chaining-explained)
+ [AngularJS Promises - The Definitive Guide](http://www.dwmkerr.com/promises-in-angularjs-the-definitive-guide/)
--->
