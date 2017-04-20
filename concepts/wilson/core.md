## Wilson Core

Each app that uses Wilson has a global "wilson" instance that is decorated onto the window object.
This is where your primary interfacing with Wilson will take place. To the developer, the global "wilson" 
instance should feel like a simple wrapper for angularjs declarations and a means of utility, logging, 
and config access (with a few exceptions for advanced use cases).


## Wilson Interface Methods

* [setAppConfig](#setappconfig)
* [getActivePage](#getactivepage)
* [getActiveComponent](#getactivecomponent)
* [getActiveComponentList](#getactivecomponentlist)
* [component](#component)
* [behavior](#behavior)
* [service](#service)
* [factory](#factory)
* [resource](#factory)
* [class](#class)
* [utility](#utility)
* [filter](#filter)


## Wilson Properties

* [config](#config)
* [utilities](../utilities/utilities.md)
* [log](../logging/logging.md)


A detailed typescript definition can be found [here](https://raw.githubusercontent.com/hightail/wilson/master/wilson.d.ts). Note: this file can be referenced in your IDE to provide
syntax highlighting and auto-completion for wilson projects.


### setAppConfig

Set the wilson instance config to a given object. This method takes a plain javascript object and sets it
as the config. Read more about the wilson config [here](#config). 

```typescript
function setAppConfig(config: Object): void;
```
Usage Example:
```js
wilson.setAppConfig({
  app: {
    version:    '1.0.0',
    name:       'MyApp',
    mountpath:  '/wilson'
  }
});
```


### getActivePage

Returns the name of the currently active page component. Effectively, which page are you on.

```typescript
function getActivePage(): string;
```


### getActiveComponent

Return relevant object references and information for a given active wilsonComponentId. The returned
object contains a stored reference to $scope and controller for the given wilsonComponentId. This is useful
for tying $element references back to our components without needing to use angularjs's [$compileProvider.debugInfoEnabled](https://docs.angularjs.org/guide/production) option.

```typescript
function getActiveComponent(componentId: string): Object;
```
Usage Example:
```js
// Assume we've have an instance of a component with a full name of 'ht-my-component' in the DOM

function someHandler() {
  var componentId = $('.ht-my-component').data('wilsonComponentId');
  var component   = wilson.getActiveComponent(componentId);
  
  console.log(component.name);        // --> The simple name of our component "my-component"
  console.log(component.scope);       // --> The isolateScope instance for this component
  console.log(component.controller);  // --> The controller instance for this component
}

```


### getActiveComponentList

Returns a list of all active component objects on wilson.

```typescript
function getActiveComponentList(): Object[];
```


### component

Declare a new component onto the wilson app module. This method is simply a wrapper for declaring a
"component-style" directive onto angularjs. Wilson components are intended to have isolateScope and
and explicit controller (which is ensured by wilson upon declaration). 

In terms of the config, all standard angularjs directive configuration properties are supported, please 
see the angularjs [directive api](https://docs.angularjs.org/api/ng/service/$compile). More specifics of 
Wilson components can be found [here](../components/components.md).

```typescript
function component(name: string, config: Object): void;
```
Usage Example:
```js
wilson.component('my-component', {
  controller: ['$scope', '$element', function($scope, $element) {
    
  }]
});
```


### behavior

Declare a new behavior onto the wilson app module. This method is simply a wrapper for declaring an
"attribute-style" directive onto angularjs. There are no restrictions or assumptions made for wilson
behaviors, all standard angularjs directive configuration properties are supported, please see the 
angularjs [directive api](https://docs.angularjs.org/api/ng/service/$compile). 

```typescript
function behavior(name: string, definition: any[]|Function): void;
```
Usage Example:
```js
wilson.behavior('my-behavior-directive', ['$window', function($window) {
  return {
    link: function($scope, $element, $attrs, controller) {
      $window.alert('my-behavior-directive has been activated!');
    }
  };
}]);
```