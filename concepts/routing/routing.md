# Wilson Routing

Routing is a key area where wilson provides extended functionality to angular applications. In angularjs, routes are 
defined from a URL path to a controller and a template/templateUrl. This requires that all templates
and controllers be declared on angular prior to routing. With wilson, routes are defined from a url path to a wilson
component (angularjs element-style directive). At routing time, wilson determines all necessary dependencies needed to fulfill
the route, loads all javascript, markup templates and styles then resolves the route and renders the routed component.

This gives wilson applications the ability to "code-split" their javascript, markup templates and styles. Although the application 
is a SPA, it does not need to load a monolithic javascript payload that contains the entire application. Rather, as the user navigates
to different pages, new javascript is added to support the new areas. This greatly reduces the payload size of javascript needed
to initially load the application and removes development concern as a codebase grows larger and larger over time.

Routing is dependent on the use of a wilson component as the target of any given route. For more information on wilson
components see the [detailed wilson component api](../components/components.md)

> Under-the-hood, wilson uses the standard angularjs [ngRoute](https://docs.angularjs.org/api/ngRoute) module. For more
> detailed information about angularjs routing see the [$routeProvider documentation](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider).
  
# Route Definition

All routes for wilson apps must be declared in a external json file. By default the wilson server application looks for
routes in "/client/routing.json" (this is configurable on the server via the property **server.projectPaths.routes**). These
routes are read by the server and decorated onto the [wilson.config](../wilson/core.md#wilson-config) object that is sent to the client.

Example routing.json
```json
{
  "routes": [
    {
      "path":           "/",
      "component":      "home",
      "title":          "Home Page",
      "options":        {}
    },
    {
      "path":           "/dashboard/:tabId",
      "component":      "dashboard",
      "title":          "Dashboard Page",
      "preload":        true,
      "options":        {}
    },
    {
      "path":           "/dashboard/news",
      "component":      "dashboard",
      "title":          "Dashboard News Feed",
      "defaultParams":  { "tabId": "news" },
      "options":    {}
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

It is important to note that this file must include a null route as its final route entry. This is used as a catch-all
for any non-matched route (effectively 404 Not Found). Wilson will throw an error upon bootstrapping if it does not find
a null route in the config or if the null route is not the final route entry.


# Route Entries

Each route entry supports the following distinct properties:

|     Property     |   Type   |     Description      |
| ---------------- |:--------:| -------------------- |
| **path**           | string   | The URL path, see angularjs [$routeProvider.when()](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider#when) |
| **component**      | string   | The name of the target component for this route |
| **title**          | string   | The html title to use for this page (i.e. <title>...</title>) |
| **preload**        | boolean  | A flag to specify if the contents of this route should be pre-loaded - Optional |
| **defaultParams**  | Object   | An object containing default route parameters that should be applied - Optional |
| **options**        | Object   | An object with optional data used to provide extra functionality |


These are the properties that wilson reads and uses from the route entries. Without adding any option support, an
application can perform simple routing from page to page with changing titles. 

> Note that **title** may be templated with dynamic content using the [translateTitle()](#translateTitle) method, covered below.

The **preload** property allows designated route content to be pre-loaded in order to improve user experience. If preload is enabled for a
route, it's content will be fetched after the first $locationChangeSuccess event is fired when the app is loaded. All component js, markup
and styles will be loaded silently loaded behind the scenes so that when those routes are eventually navigated to they will already
be cached and can be fulfilled immediately.

The **options** property provides a blank canvas for applications to build intricate routing functionality. Using these 
properties in conjunction with the IRouteService interface, developers can create new routing constructs as they desire.
 
# Implementing a custom router

Wilson applications may implement a custom router to handle special routing functionality. The router is a service that is used at specific
points during wilson routing to determine how/when to proceed with route fulfillment. This allows the application to control
things like restricted routes, path forwarding, dependency resolution, session-based redirections and much more.

The custom router service has access to all route information including the options and data for the current route, this allows distinctive
configuration per-route that allows for intelligent handling.

Creating a custom router is as easy as configuring it's location in [wilson-config.json](../wilson/core.md#wilson-config) and then declaring it using **wilson.router**. The 
default configured location is **/client/src/router.js**. The router definition is effectively the same as a service definition with the only
difference being that there is no service name needed for the router.

Example route entry with options: 
```json
{
  "routes": [
    {
      "path":       "/user-profile/:userId",      
      "component":  "user-profile",
      "title":      "Profile",          
      "options": {
        "loggedOutRedirect":  "/login"            
      }
    }
  ]
}
```

Note the **loggedOutRedirect** options property. Let's assume this property should do exactly what it sounds like it should do: 
redirect a user without a session to "/login". This functionality can easily be implemented via IRouteService's 
[handleRouteChange()](#handleRouteChange) method.

Example handling in a custom router:
```js
wilson.router(['$q', '$location', 'AuthService', 
  function($q, $location, AuthService) {
    
    function handleRouteInfo(currentRoute, routeOptions, routeInfo) {
      
      // If user is not logged in and there is a loggedOutRedirect defined, redirect
      if (routeOptions.loggedOutRedirect && !AuthService.isLoggedIn()) {
        $location.replace();
        $location.path(routeOptions.loggedOutRedirect);    // --> redirect to "/login" (from case above)
        return $q.reject();
      }
      
      return $q.when();
    }
    
    // Return service
    return { handleRouteInfo: handleRouteInfo };
  }
]);
```

As you can see in the example, the **handleRouteChange** method has access to routeOptions (the options object 
from the current route). The custom router can inject other application services and use custom logic to support
any number of specialized functionalities like the one above.


# Custom Router Interface

* [handleRouteChange](#handleRouteChange)
* [loadDependencies](#loadDependencies)
* [loadSession](#loadSession)
* [getTitleText](#getTitleText)

> NOTE: A custom router acts as an extension of the default router and may implement one
> or more of the above interface methods. Interface methods not included in the custom router
> will inherit the default functionality.

## <a name="handleRouteChange"></a>handleRouteChange(currentRoute, routeOptions, routeInfo)

Handles application logic before a route change is fulfilled. Returns a promise that
determines whether routing may continue. A resolved promise means routing should
continue normally, a rejected promise will stop the current route from completing.

This method accepts three parameters:

* *currentRoute* - The URL path of the current route
* *routeOptions* - All data and options properties from the current route
* *routeInfo*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- The full route config entry for the current route  

```typescript
function handleRouteChange(currentRoute: string, routeOptions: Object, routeInfo: Object): Promise;
```
Implementation Example:
```js
wilson.router(['$q', '$location', 
  function($q, $location) {
    
    function handleRouteInfo(currentRoute, routeOptions, routeInfo) {
 
      if (routeOptions.forward) {
        $location.replace();
        $location.path(routeOptions.forward);
        return $q.reject();
      }
      
      return $q.when();
    }
    
    // Return service
    return { handleRouteInfo:  handleRouteInfo };
  }
]);
```


## <a name="loadDependencies"></a>loadDependencies(routeInfo)

Handles the loading of any required dependencies before a route change is fulfilled. Returns a promise that
determines whether routing may continue. A resolved promise means routing should continue normally, a rejected promise
will stop the current route from completing.

This method accepts one parameter:

* *routeInfo* - The full route config entry for the current route
 
```typescript
function loadDependencies(routeInfo: Object): Promise;
```
Implementation Example:
```js
wilson.router(['$q', 
  function($q) {
    
    function loadDependencies(routeInfo) {
      var promises = [];
      
      // Iterate over an optional special property "jsDependencies" (a list of required js scripts)
      _.each(routeInfo.options.jsDependencies, function(scriptPath) {
        var jsPromise = $q.defer();
 
        $.getScript(scriptPath).done(jsPromise.resolve).fail(jsPromise.reject);
         
        promises.push(jsPromise.promise);
      });
      
      return $q.all(promises);    // Resolves only if all script promises are resolved 
    }
    
    // Return service
    return { loadDependencies:  loadDependencies };
  }
]);
```

## <a name="loadSession"></a>loadSession()

Handles the loading of any session-related data before a route change is fulfilled. Returns a promise that
determines whether routing may continue. A resolved promise means routing should continue normally, a rejected promise
will stop the current route from completing.

```typescript
function loadSession(): Promise;
```
Implementation Example:
```js
wilson.router(['$q', 'AuthService', 
  function($q, AuthService) {
    
    function loadSession() {
      
      if (!AuthService.hasSession()) {
        return AuthService.provisionSession();    // Returns a promise to create a session
      }
      
      return $q.when();   // We already have a session so we can resolve
    }
    
    // Return service
    return { loadSession:  loadSession };
  }
]);
```

## <a name="getTitleText"></a>getTitleText(routeTitle)

Handles the creation of the title text that will be used for the current route. Returns a string that will be used
as the page title for the route. This method may be used to transform, template or internationalize title text into a 
finalized string.

> Note: Unlike the other methods, getTitleText is synchronous and must return a string.

```typescript
function getTitleText(routeTitle: string): string;
```
Implementation Example (Basic):
```js
wilson.router([ 
  function() {
    
    function getTitleText(routeTitle) {
      return routeTitle;  // Just return the title string included in the route entry                
    }
    
    // Return service
    return { getTitleText:  getTitleText };
  }
]);
```
Implementation Example (Advanced):
```js
wilson.router(['$route',
  function($route) {
    
    function getTitleText(routeTitle) {
      var titleText = 'Default Title Here';
      
      // Attempt to template the routeTitle using lodash 
      try { titleText = _.template(routeTitle)($route.current.params); } catch(e);
      
      return titleText;                  
    }
    
    // Return service
    return { getTitleText:  getTitleText };
  }
]);
```