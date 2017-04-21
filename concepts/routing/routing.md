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

Routing is depends on the use of a wilson component as the target of any given route. For more information on wilson
components see the [detailed wilson component api](../components/components.md)
  
# Route Definition

All routes for wilson apps must be declared in a external json file. By default the wilson server application looks for
routes in "/client/routing.json" (this is configurable on the server via the property **server.projectPaths.routes**). 

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

