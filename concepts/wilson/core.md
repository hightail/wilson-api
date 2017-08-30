# Wilson Core

Each app that uses Wilson has a global "wilson" instance that is decorated onto the window object.
This is where the app's interfacing with Wilson will take place. To the developer, the global "wilson" 
instance should feel like a simple wrapper for angularjs declarations and a means of utility, logging, 
and config access. As an added convenience, wilson decorates angular's $rootScope instance with a few 
convenience/utility methods. 

[See Wilson $rootScope utilities](#wilson-rootscope-utilities) 

> A detailed typescript definition of wilson can be found [here](https://raw.githubusercontent.com/hightail/wilson/master/wilson.d.ts). NOTE: this file can be referenced in your 
> IDE to provide syntax highlighting and auto-completion for wilson projects.


Wilson Interface
====

* [setAppConfig](#setAppConfig)
* [getActivePage](#getActivePage)
* [getActiveComponent](#getActiveComponent)
* [getActiveComponentList](#getActiveComponentList)
* [findComponentId](#findComponentId)
* [destroyComponent](#destroyComponent)
* [routeInfo](#routeInfo)
* [router](#router)
* [component](#component)
* [behavior](#behavior)
* [service](#service)
* [factory](#factory)
* [resource](#resource)
* [class](#class)
* [utility](#utility)
* [filter](#filter)
* [config](#wilson-config)
* [utilities](../utilities/utilities.md)
* [log](../logging/logging.md)
  
  
## <a name="setAppConfig"></a>setAppConfig(config)

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



## <a name="getActivePage"></a>getActivePage()

Returns the name of the currently active page component. Effectively, which page are you on.

```typescript
function getActivePage(): string;
```


## <a name="getActiveComponent"></a>getActiveComponent(componentId)

Return the component $scope for a given active component instance id (wilsonComponentId). This is useful
for tying $element references back to our components without needing to use angularjs's [$compileProvider.debugInfoEnabled](https://docs.angularjs.org/guide/production) option.

```typescript
function getActiveComponent(componentId: string): Object;
```
Usage Example:
```js
// Assume we've have an instance of a component with a full name of 'ht-my-component' in the DOM

function someHandler() {
  var componentId = $('.ht-my-component').data('wilsonComponentId');
  var scope       = wilson.getActiveComponent(componentId);
  
  console.log(scope);                               // --> The isolateScope instance for this component
  console.log(componentId === scope.component.id);  // --> This should always be true
}

```


## <a name="getActiveComponentList"></a>getActiveComponentList()

Returns a list of all active component $scopes on wilson.

```typescript
function getActiveComponentList(): Object[];
```


## <a name="findComponentId"></a>findComponentId(jqElement)

Find the component that is nearest to the given jqElement and return its instance id. This method traverses up the 
DOM tree to the parent to find the component, returning null if not found.

```typescript
function findComponentId(jqElement: Array): string;
```
Usage Example:
```js
// Assume we've have an instance of a component with a full name of 'ht-my-component' in the DOM

function someHandler() {
  var someChildElement = $('.ht-my-component div.something');
  var componentId      = wilson.findComponentId(someChildElement);
  var scope            = wilson.getActiveComponent(componentId);
  
  console.log(scope);                               // --> The isolateScope instance for the "my-component" component
  console.log(scope.component.name);                // --> my-component
  console.log(componentId === scope.component.id);  // --> This should always be true
}

```


## <a name="destroyComponent"></a>destroyComponent(componentId)

Forcibly destroy an active wilson component with the given instance id (componentId).

```typescript
function destroyComponent(componentId: string): void;
```
Usage Example:
```js
// Assume we've have an instance of a component with a full name of 'ht-my-component' in the DOM

function someHandler() {
  var componentId = $('.ht-my-component').data('wilsonComponentId');
  var scope       = wilson.getActiveComponent(componentId);
  
  console.log(scope);                               // --> The isolateScope instance for the "my-component" component
  console.log(scope.component.name);                // --> "my-component"
  console.log(componentId === scope.component.id);  // --> This should always be true
  
  wilson.destroyComponent(componentId);
  
  console.log(scope.$$destroyed);                   // --> This is now true (scope is destroyed)
}

```


## <a name="routeInfo"></a>routeInfo

An object representing the route information for the currently active route. This can be used from components to
 determine any data parameters that are represented in the route information.

```typescript
routeInfo: Object;
```

Given the following routing.json:
```json
{
  "routes": [
    {
      "path":           "/dashboard/:tabId",
      "component":      "dashboard",
      "title":          "Dashboard Page",
      "preload":        true,
      "options":        {}
    },
    {
      "path":       null,
      "component":  "404",
      "title":      "Not Found",
      "options":    {}
    }
  ]
}
```

Let's say we've just routed to the "/dashboard/newsfeed" and the "dashboard" component constructor is called. The following
dashboard component implementation shows how the route parameters can be accessed via wilson.routeInfo: 

```js
wilson.component('dashboard', {
  controller: ['$scope', '$element', function($scope, $element) {
    
    console.log(wilson.routeInfo.tabId);      // --> "newsfeed"  -- Routing data is accessible here via the routeInfo object on wilson
    
  }]
});
```

> NOTE: The routeInfo object is a virtual property and represents a copy of the currently active routeInfo. This
> object is immutable and cannot be changed by its accessor. **IMPORTANT:** This means if wilson.routeInfo is stored into a local
> variable it will not update on route change. 


## <a name="router"></a>router(definition)

Declare a [custom router service](../routing/routing.md#implementing-a-custom-router) on the wilson app module. The definition is exactly the same as if it
was declared as a factory-style service on angularjs directly (sans the name parameter). See the angularjs documentation for [$provide.factory](https://docs.angularjs.org/api/auto/service/$provide#factory). 

```typescript
function router(definition: any[]|Function): void;
```
Usage Example:
```js
wilson.router(['$q', 
  function($q) {
    
    function handleRouteInfo(currentRoute, routeOptions, routeInfo) {
      return $q.when();
    }
    
    // Return service
    return { handleRouteInfo: handleRouteInfo };
  }
]);
```


## <a name="component"></a>component(name, config)

Declare a new component onto the wilson app module. This method is a wrapper for declaring a
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


## <a name="behavior"></a>behavior(name, definition)

Declare a new behavior onto the wilson app module. This method is a wrapper for declaring an
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


## <a name="service"></a>service(name, definition)

Declare a new service onto the wilson app module. This method is wrapper for declaring
factory-style services on angularjs. Definitions for services are exactly the same as if they
were declared on angularjs directly. See the angularjs documentation for [$provide.factory](https://docs.angularjs.org/api/auto/service/$provide#factory). 

```typescript
function service(name: string, definition: any[]|Function): void;
```
Usage Example:
```js
wilson.service('AuthService', ['$http', function($http) {
  return {
    login: function(email, password) {
      return $http.post('/api/login', { email: email, password: password });
    }
  };
}]);
```


## <a name="factory"></a>factory(name, definition)

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function factory(name: string, definition: any[]|Function): void;
```


## <a name="resource"></a>resource(name, definition)

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function resource(name: string, definition: any[]|Function): void;
```


## <a name="class"></a>class(name, definition)

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function class(name: string, definition: any[]|Function): void;
```


## <a name="utility"></a>utility(name, definition)

*Alias* for **service()**. Allows semantic distinction between the service types of a wilson app.

```typescript
function utility(name: string, definition: any[]|Function): void;
```


Wilson Config
====

An application using wilson is required to provide a configuration object to the wilson instance. This
config object contains properties that represent version, selector prefixes, mountpath for the wilson server
application and other values that allow each application to use wilson flexibly.

There are 4 required sections of this config from wilson's perspective:

| Section          | Description                   |
| ---------------- | ----------------------------- |
| **app**          | *Core configuration settings* | 
| **routes**       | *Application URL routes*      | 
| **tags**         | *Client identity info*        |
| **i18n**         | *i18next configuration*       |


It is recommended that apps using wilson adopt the wilson.config object as their primary client-side
app config. Any config-type data that an application requires during runtime can be designated onto this 
config up-front, and will then be accessible via the global wilson instance from anywhere.

The wilson server instance ultimately sets most of the values on the config upon serving it to the browser. Although
there is a method available to [setAppConfig](#setAppConfig) directly on the wilson instance, the config is pre-set when the config
module is fetched from the server (required as the first step of wilson bootstrapping).

Example Config:

```json
{
  "app": {
    "name":               "MyApplication",
    "version":            "1.0.0",
    "connectionFilters":  "EYJwhgdgJgvAzgBzAYwKZwD6gPYHc6ogzIAWI2AtqhlKgG4CWaMFKGANpAOYCuYXqGKggYEnAC4AzbCAoxacANbjsCIA"
    "assetPath":          "/client",
    "useVersionedAssets": true
  },
  "routes": [
    {
      "path":       "/",
      "component":  "home",
      "title":      "Home Page",
      "options":    {}
    },
    {
      "path":       null,
      "component":  "404",
      "title":      "Not Found",
      "options":    {}
    }
  ],
  "tags": {
    "brand":      "my-app",
    "browser":    "chrome",
    "device":     "mac",
    "platform":   "desktop",
    "language":   "en"
  },
  "i18n": {
    "clientSafeNamespaces":         "all",
    "defaultLng":                   "en",
    "fallbackLng":                  "en",
    "lng":                          "en",
    "localStorageExpirationTime":   604800000,
    "namespaces":                   [],
    "resStore":                     {},
    "sendMissing":                  true,      
    "supportedLngs":                ["en"],
    "useLocalStorage":              false
  }
}
```

## App Config

App settings are mostly descriptive, including the name and version of the app. The one notable setting here 
is the **connectionFilters** property.

**connectionFilters** is a hash of the keys/values in the **tags** section of the config. Specifically the server uses the
[lz-string](https://github.com/pieroxy/lz-string) library's [compressToBase64()](http://pieroxy.net/blog/pages/lz-string/guide.html#inline_menu_4) method
to create a hash that can be appended to server request URLs for component bundles (this allows the wilson server to serve back templates
based on the user's **tags**. NOTE: This is also vital for caching when using a CDN).

Code-based Explanation:
```js

var connectionFilters = wilson.config.app.connectionFilters;
var rawFilters        = LZString.decompressFromBase64(connectionFilters);
var tags              = wilson.config.tags;

console.log(connectionFilters);   // --> "EYJwhgdgJgvAzgBzAYwKZwD6gPYHc6ogzIAWI2AtqhlKgG4CWaMFKGANpAOYCuYXqGKggYEnAC4AzbCAoxacANbjsCIA"
 
console.log(rawFilters);          // --> "brand=spaces|browser=chrome|device=mac|language=en|platform=desktop"

console.log(tags.brand);          // --> "spaces"
 
console.log(tags.browser);        // --> "chrome"
 
console.log(tags.device);         // --> "mac"
 
console.log(tags.platform);       // --> "desktop"
 
console.log(tags.language);       // --> "en"
```


## Route Config

All URL routes for an application are defined on wilson.config.routes. At bootstrap time, wilson declares all routes onto 
angular using the [ngRoute module](https://docs.angularjs.org/api/ngRoute). Similar to routing in the new angular.io, route entries on wilson map a URL "path" 
to a "component". 

#### [Detailed Wilson routing api](../routing/routing.md)


## Tags Config

The tags properties in the wilson config represent identifying information for the currently active client. This info 
includes device-type, platform, browser, language and even branding. This means that client wilson applications do not 
have to use javascript tactics to detect these characteristics, but can simply imply them from the global wilson config.


## i18n Config

The i18n settings follow the [i18next](https://github.com/i18next/i18next/tree/2.2.0) library configuration. Wilson provides 
i18n support via the use of this library both server-side and client-side. On the wilson server, i18next is used to pre-internationalize
markup templates before they make it to the client browser. On the client, wilson provides translation methods for dynamic
strings that are templated during runtime of the application.



Wilson $rootScope Utilities
====

- [triggerDigest](#triggerDigest)
- [bindToDigest](#bindToDigest)


## <a name="triggerDigest"></a>triggerDigest()

Triggers an angular digest cycle. This method will not throw an exception if a digest is in progress (in the event that a
digest is in progress, another will be triggered). This method returns a promise that will be called after the digest.
 
```typescript
function triggerDigest(): Promise;
```

Usage Example:
```js
$rootScope.message = 'Hello';     // --> Set message
    
function updateMessage(msg) {
  $rootScope.message = msg;
}
    
setTimeout(function() {
  updateMessage('World');      // --> Function updates message but will not trigger a digest automatically
  $rootScope.triggerDigest();  // --> Triggers the digest cycle and the updated message is now propagated to the view  
}, 2000);
```

## <a name="bindToDigest"></a>bindToDigest(method, context)

Binds a given function to a digest cycle trigger. This method returns a new function that invokes the original function 
and then triggers a digest cycle. This is useful if for out-of-context event handlers that update bound data references.

```typescript
function bindToDigest(method: Function, context: any): Function;
```

Usage Example:
```js
$rootScope.message = 'Hello';
 
var updateMessage = $rootScope.bindToDigest(function(msg) {
  $rootScope.message = msg;
});
 
setTimeout(function() {
  updateMessage('World');      // --> Updates message and triggers a digest cycle - changes are propagated to the view 
}, 2000);

```