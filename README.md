# Wilson - v3.1.0

Wilson is framework that adds a distinct component abstraction to Angularjs web applications. These components 
can be lazily-loaded at route-time automatically based on dependencies (this works in conjunction with the Wilson
server). These components are decorated with common base-functionality that abstracts memory management for eventing
and other various conveniences that empower developers and reduce boiler-plate code. 

This document provides details around the different concepts in Wilson and their distinctive interfaces. Sections
are broken up into different pages which contain more detailed documentation.

## Wilson Basics

When using Wilson, all Angularjs constructs are created via a Wilson declaration.

E.G.
```
wilson.component('my-component',  [..., function(...) { }]);
 
wilson.behavior('my-behavior',    [..., function(...) { }]);
 
wilson.service('MyService',       [..., function(...) { }]);
 
wilson.resource('MyResource',     [..., function(...) { }]);
 
wilson.utility('MyUtility',       [..., function(...) { }]);
 
wilson.class('MyClass',           [..., function(...) { }]);
 
wilson.filter('MyFilter',         [..., function(...) { }]);
```

Although this initial declaration is not specifically on the 'angular' global instance, under-the-hood, wilson
is creating these constructs (or the appropriate construct) on angular for these objects.  It is important to note
that the definitions inside of these method calls are in fact purely angular definitions and support all standard
angular functionality. There are a few conveniences added by Wilson, but there are no removals of angular concepts.

To go into a few more specifics, here are the relationships between what is declared on Wilson and what is created
on angular:

```
wilson.component() ---> angular.directive()
wilson.behavior()  ---> angular.directive()
  
wilson.service()   ---> angular.factory()
wilson.resource()  ---> angular.factory()
wilson.class()     ---> angular.factory()
wilson.utility()   ---> angular.factory()
  
wilson.filter()    ---> angular.filter()
```

Wilson ensures that some base options provided to angular so that these constructs are represented appropriately, but as
described above, Wilson is simply declaring these onto angular and also providing the support to accept new declarations
as lazily-loaded javascript content is added during navigation.


## Wilson Concepts


1. [Core Module](./concepts/wilson/core.md)
2. [Config](./concepts/wilson/core.md#wilson-config)
3. [Component Interface](./concepts/components/components.md)
4. [Routing](./concepts/routing/routing.md)
5. [Utility Functions](./concepts/utilities/utilities.md)
6. [Logging](./concepts/logging/logging.md)