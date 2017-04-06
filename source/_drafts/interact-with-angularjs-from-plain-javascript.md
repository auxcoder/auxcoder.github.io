---
layout: post
title: Interact with AngularJS from plain JavaScript
comments: false
category: Coding
tags:
- javascript
- angularjs
- DOM
author: auxcoder
---

**Selecting by DOM element**

```js
angular.element(domElement).scope() // to get the current scope for the element
angular.element(domElement).injector() // to get the current app injector
angular.element(domElement).controller() // to get a hold of the ng-controller instance.
```

Any changes to the angular model or any method invocations on the scope need to be wrapped in `$apply()` like this:

```js
// from inside controller
vm.$apply(function(){
  // perform any model changes or method invocations here on angular app.
});
```

**Access and change variable in scope**

```js
angular.element(domElement).scope().myVar = 5;
angular.element(myDomElement).scope().myArray.push(newItem);
// Update page to reflect changed variables
angular.element(myDomElement).scope().$apply();
```

### Applying change with data from outside angular
**Applying controller method from outside**

```js
// angular controller
angular.module('app').controller('MyCtrl', ['$log', function(log){
	scope.myMethod = myMethod;

	function myMethod(argument) {
		$log.debug('This is' + argument);
	}
}])

// data outside angular
val = {data: 'some data'};

// get an element inside the target controller scope
elm = document.getElementById('myAngularApp');

// accessing the scope
scope = angular.element(elm).scope();

// applying changes
scope.$apply(function() {
	scope.myMethod(val);
});

```

### Accesing to a controller function from outside of angular

**Access to exiting controller function**

```js
function incrementFromOutside(){
	document.getElementById('myAngularApp').scope().incrementDataInService()
}

// angular controller
function myController($scope, myService) {
    $scope.x = 1;
    $scope.incrementDataInService= function() {
        x++;

        // if you are calling from legacy code, you will need to invoke $apply()
        $scope.$apply();
    }
}
```

+ [Sources](http://stackoverflow.com/questions/10490570/call-angular-js-from-legacy-code)
+ [Examples](http://jsfiddle.net/peterdrinnan/2nPnB/16/)
