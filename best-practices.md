# Wilson Best Practices

There are a handful of recommended practices for wilson apps when it comes to using different constructs.
These practices help make the code more self-documenting and organized. Ultimately they are meant to
aid developers as applications grow in size and complexity.

## Components

**1 *Definitions should follow the same property ordering for each component***

This is the recommended pattern:

```js
wilson.component('my-component', {
  transclude: true,                                           // Optional
  scriptDependencies: [],                                     // Optional    
  scope: {},                                                  // Optional
  exports: {},                                                // Optional
  inherit: {},                                                // Optional
  require: [],                                                // Optional
  controller: [function() {}],                                // Required                         
  link: function($scope, $element, $attrs, controller) {}     // Optional
});
```
> There are more options available for components, but these are the most common and should
> maintain this recommended ordering.

___

**2 *Avoid using "this" in a component controller. Use private functions or declare them on the $scope so that they can be used in from the view.***

Wilson components treat $scope as the effective controller. All controller functions should be declared privately or
publicly (on the $scope instance). This keeps consistency in the pattern that $scope represents a component instance.


```js
wilson.component('my-component', {
  controller: ['$scope', function($scope) {

    function foo = function foo(msg) {  // GOOD - Explicit private function only available inside this controller
        alert(msg);
    };

    $scope.bar = function bar(msg) {    // GOOD - Explicit $scope function available in the controller and view
        alert(msg);
    };

    this.foobar = function foobar(msg) {      // BAD - "this" is on the controller which is not used
        alert(msg);
    };

  }]
});
```
___

**3 *Declare explicit function names for all $scope functions***

This makes debugging much easier for the developer. If an error prints to the console, the 
stack trace will have a function name to point out where the problem occurred. Without it the 
console will print "Anonymous Function" which is ambiguous and does not help with troubleshooting.

```js
wilson.component('my-component', {
  controller: ['$scope', function($scope) {

    $scope.message = '';
    
    $scope.updateMessage = function updateMessage(message) {        // GOOD - Function name matches the property name
      $scope.message = message;
    };
    
    $scope.clearMessage = function() { $scope.message = ''; };      // BAD  - Function has no name
  }]
});
```

## Services

**1 *Service object interfaces should be at the bottom of each service***

This make the public interface for any service very easy to find. When a developer is looking for a function 
inside of a service, they can simply look at the bottom of the file to find all methods that are publicly 
available for use.

**Good**
```js
wilson.service('MyService', ['$rootScope', function($rootScope) {
  
  /*** Service Methods ***/
  function doSomething() {}
  
  function doSomethingElse() {}
  
  function doAllTheThings() {}
  
  
  /*** Service Public Interface ***/
  var service = {
    doSomething:      doSomething,
    doSomethingElse:  doSomethingElse,
    doAllTheThings:   doAllTheThings
  };
  
  return service;
}]);
```
**Not Recommended**
```js
wilson.service('MyService', ['$rootScope', function($rootScope) {
  var service = {};
  
  /*** Service Methods ***/
  service.doSomething     = function doSomething() {};
  
  service.doSomethingElse = function doSomethingElse() {};
  
  service.doALlTheThings  = function doAllTheThings() {};
  
  return service;
}]);
```