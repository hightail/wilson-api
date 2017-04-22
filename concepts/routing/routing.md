# Wilson Routing

Routing is a key area where wilson provides extended functionality to angular applications. In angularjs, routes are 
defined from a URL path to a controller and a template/templateUrl. This requires that all templates
and controllers be declared on angular prior to routing. With wilson, routes are defined from a url path to a wilson
component (angularjs element-style directive). At routing time, wilson determines all necessary dependencies needed to fulfill
the route, loads all javascript and markup templates and then resolves the route and renders the routed component.

This gives wilson applications the ability to "code-split" their javascript and markup templates. Although the application 
is a SPA, it does not need to load a monolithic javascript payload that contains the entire application. Rather, as the user navigates
to different pages, new javascript is added to support the new areas. This greatly reduces the payload size of javascript needed
to initially load the application and removes development concern as a codebase grows larger and larger over time.

Routing is dependent on the use of a wilson component as the target of any given route. For more information on wilson
components see the [detailed wilson component api](../components/components.md)

> Under-the-hood, wilson uses the standard angularjs [ngRoute](https://docs.angularjs.org/api/ngRoute) module. For more
> detailed information about angularjs routing see the [$routeProvider documentation](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider).
  
## Route Definition

All routes for wilson apps must be declared in a external json file. By default the wilson server application looks for
routes in "/client/routing.json" (this is configurable on the server via the property **server.projectPaths.routes**). These
routes are read by the server and decorated onto the [wilson.config](../wilson/core.md#wilson-config) object that is sent to the client.

Example routing.json
```json
{
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
  ]
}
```

It is important to note that this file must include a null route as its final route entry. This is used as a catch-all
for any non-matched route (effectively 404 Not Found). Wilson will throw an error upon bootstrapping if it does not find
a null route in the config or if the null route is not the final route entry.


## Route Entries

Each route entry has 4 distinct properties:

|     Property     |   Type   |     Description      |
| ---------------- |:--------:| -------------------- |
| **path**         | string   | The URL path, see angularjs [$routeProvider.when()](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider#when) |
| **component**    | string   | The name of the target component for this route |
| **title**        | string   | The html title to use for this page (i.e. <title>...</title>) |
| **options**      | Object   | An object with optional data used to provide extra functionality |


These are the basic properties that wilson reads and uses from the route entries. Without adding any option support, an
application can perform simple routing from page to page with changing titles. Note that **title** may template in
any params in the **path** property using [lodash templating syntax](https://lodash.com/docs/4.17.4#template).

The **options** property provides a blank canvas for applications to build intricate routing functionality. Using these 
properties in conjunction with the IRouteService interface, developers can create new routing constructs as they desire.
 
## IRouteService Interface

Every wilson application is required to implement a special service call IRouteService. This service is used at specific
points during wilson routing to determine how/when to proceed with route fulfillment. This allows the application to control
things like restricted routes, path forwarding, session-based redirections and much more. IRouteService