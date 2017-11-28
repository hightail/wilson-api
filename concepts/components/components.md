# Wilson Components

Components are the heart of a wilson application. They are custom elements that contain a subset
of functionality that can be re-used as needed. Components can represent a page (the target 
of a route) or a building-block that is used to as a distinct piece of a user interface. 

There are 3 important pieces that make up a wilson component:

* *The template*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- The HTML markup that defines the content (including sub-component usage)
* *The controller*&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- The javascript "control" (code-behind) for the template
* *The style definition*&nbsp;- The visual treatment of the component markup

Example component:

**nav-bar.html**
```html
<div class="nav-bar">
  <ul class="tabs">
    <li ng-repeat="opt in navOptions" class="tab" ng-click="navigate(opt.target)">[[opt.name]]</li>
  </ul>
</div>
```
**nav-bar.js**
```js
wilson.component('nav-bar', {
  controller: ['$scope', '$location', function($scope, $location) {
    
    $scope.navOptions = [
      { name: 'Home',       target: '/home'       },
      { name: 'Dashboard',  target: '/dashboard'  },
      { name: 'Settings',   target: '/settings'   }
    ];
    
    $scope.navigate = function navigate(targetPath) {
      $location.path(targetPath);
    };
    
  }]
});
```
**nav-bar.scss**
```scss
.nav-bar {
  position: fixed;
  top: 0; right: 0; left: 0;
  height: 50px;
  
  ul {
    li { display: inline-block; }
  }
}
```

> NOTE: In the markup template notice the "[[]]" syntax. Wilson changes the interpolation symbols to square brackets instead
> of braces. This is to support internationalization templating on the backend. If you run into issues with templated content
> Just remember that all bindings should use square brackets in order to work in wilson.

Under-the-hood, a component is just an angularjs "element-style" directive that has isolateScope and defines
an explicit controller ([see angularjs components](https://docs.angularjs.org/guide/component)). 

So what makes wilson components different than just using angularjs directly?

Good question.

From the server, wilson provides dependency resolution for all components in the codebase. This means
that the wilson server reads and determines all dependencies between components/behaviors/services in
the application. This makes it possible to lazy-load components as they are needed when the app
navigates to a new area. No registration or importing is required by the developer, simply create
a component and start using it...Magic.

Apart from dependency resolution, wilson also applies specialized base functionality onto each component. This
ensures that each component has the same base interface and gives the developer access to
common functions and utilities, which helps to eliminate boiler plate and code duplication. Specifically, each
component's $scope is decorated with a special interface that makes it a wilson component.

[See the code in ComponentFactoryService](https://github.com/hightail/wilson/blob/4.1.2/lib/client/src/services/ComponentFactoryService.js)

# $scope decorations

* [component](#component)
* [state](#state)
* [defaultValue](#defaultValue)
* [translate](#translate)
* [triggerDigest](#triggerDigest)
* [bindToDigest](#bindToDigest)
* [stateMachine](#stateMachine)
* [storage.get](#storageGet)
* [storage.set](#storageSet)
* [on.event](#onEvent)
* [on.signal](#onSignal)
* [on.watch](#onWatch)
* [on.digest](#onDigest)
* [on.pageUnload](#onPageUnload)


## <a name="component"></a>$scope.component

An object representing the identification metadata of the component.

```typescript
component: Object;
```
Properties:
```json
{
  "id": "bd671303-2a19-4ab2-c175-9c3cb591ac09",     // Unique Id for this instance
  "name": "nav-bar"                                 // Name of the component
}
```

Usage Example:
```js
wilson.component('nav-bar', {
  controller: ['$scope', function($scope) {
    var message = 'This is the ' + $scope.component.name + ' component';
    
    wilson.log.info(message);   // --> "This is the nav-bar component"
  }]
});
````


## <a name="state"></a>$scope.state

A special stateMachine object that can be used to add state-based control to a component. Uses the 
[javascript-state-machine library](https://github.com/jakesgordon/javascript-state-machine). 

```typescript
state: Object;
```

See details below for the [stateMachine() method](#stateMachine).


## <a name="defaultValue"></a>$scope.defaultValue(scopePropertyName, defaultValue)

Assign a default value to a scope field if it is not set. Useful for optional scope bindings so that
a default value is assigned to the property.

```typescript
function defaultValue(scopePropertyName: string, defaultValue: any): any;
```
Usage Example:
```js
wilson.component('nav-bar', {
  scope: {
    "activeTab": "=?activeTab"
  },
  controller: ['$scope', function($scope) {
    
    $scope.defaultValue('activeTab', 'home');   // This sets activeTab to home if no binding is provided
    
    wilson.log.info(!!$scope.activeTab);        // --> this will always be true
  }]
});
```


## <a name="translate"></a>controller.translate(text, options)

Translate a text string using the i18next service. Used for internationalizing dynamic strings
during run-time of the application. Accepts the original text and returns a translated string.

```typescript
function translate(text: string, options: Object): string;
```
Usage Example:
```js
wilson.component('my-component', {
  controller: ['$scope', function($scope) {
    
    // Translate to Spanish
    var translatedText  = $scope.translate('Where is my dog?', { lng: 'es' });
    
    wilson.log.info(translatedText);    // --> "Donde esta mi perro?"
    
  }]
});
```


## <a name="triggerDigest"></a>$scope.triggerDigest()

Triggers an angular digest cycle. This method will not throw an exception if a digest is in progress (in the event that a
digest is in progress, another will be triggered). This method returns a promise that will be called after the digest.
 
```typescript
function triggerDigest(): Promise;
```
Usage Example:
```js
wilson.component('my-component', {
  controller: ['$scope', function($scope) {
    
    $scope.message = 'Hello';
    
    function updateMessage(msg) {     // Note this is a private non-scope method
      $scope.message = msg;
    }
        
    setTimeout(function() {
      updateMessage('World');   // --> Function updates message but will not trigger a digest automatically
      $scope.triggerDigest();   // --> Triggers the digest cycle and the updated message is now propagated to the view  
    }, 2000);
  }]
});
```

## <a name="bindToDigest"></a>$scope.bindToDigest(method, context)

Binds a given function to a digest cycle trigger. This method returns a new function that invokes the original function 
and then triggers a digest cycle. This is useful if for out-of-context event handlers that update bound data references.

```typescript
function bindToDigest(method: Function, context: any): Function;
```

Usage Example:
```js
wilson.component('my-component', {
  controller: ['$scope', function($scope) {
    
    $scope.message = 'Hello';
    
    var updateMessage = $scope.bindToDigest(function(msg) {
      $scope.message = msg;
    });
        
    setTimeout(function() {
      updateMessage('World');   // --> Updates message and triggers a digest cycle - changes are propagated to the view 
    }, 2000);
  }]
});
```

> NOTE: The examples above for triggerDigest and bindToDigest both use the setTimeout method, this is simply 
> for demonstration purposes. In practice developers should use the $timeout service which will trigger a digest
> by default


## <a name="stateMachine"></a>$scope.stateMachine(config, callback)

Setup a bindable stateMachine on the component $scope. Uses the [javascript-state-machine library](https://github.com/jakesgordon/javascript-state-machine) to create
a state-based control concept for a component. This method takes a specific configuration object unique to the 
[StateMachine.create()](https://github.com/jakesgordon/javascript-state-machine#usage) method of javascript-state-machine.

```typescript
function stateMachine(config: Object, callback: Function): void;
```
Usage Example:

```js
wilson.component('dashboard', {
  controller: ['$scope', '$timeout', function($scope, $timeout) {
    
    $scope.message = 'Please Wait...';
    
    // Setup the stateMachine
    $scope.stateMachine({
      initial: 'Loading',
      events:  [
        { name: 'loaded', from: 'Loading', to: 'Ready'  },
        { name: 'failed', from: 'Loading', to: 'Failed' }
      ],
      timeouts: [],   // this property timeouts is a wilson extended option
      callbacks: {
        onEnterReady: function() {
          $scope.message = 'Dashboard Ready';
        },
        onEnterFailed: function() {
          $scope.message = 'Dashboard Failed';
        }
      }
    });
    
    $timeout(function() {
      $scope.state.loaded();
      
      wilson.log.info($scope.state.current);        // --> "Ready"
      wilson.log.info($scope.message);              // --> "Dashboard Ready"
    });
     
    wilson.log.info($scope.state.current);          // --> "Loading"
    wilson.log.info($scope.message);                // --> "Please Wait..."
  }]
});
```
Wilson supports an extended set of options for state machines. Specifically the **timeouts** array shown in the
example above. The **timeouts** option allows for an array of time-based state transitions that are automatically
controlled by the stateMachine object itself. 

Example stateMachine definition:
```js
$scope.stateMachine({
  initial: 'Loading',
  events:  [
    { name: 'loaded', from: 'Loading', to: 'Ready'    },
    { name: 'failed', from: 'Loading', to: 'Failed'   },
    { name: 'retry',  from: 'Failed',  to: 'Loading'  }
  ],
  timeouts: [
    { state: 'Failed', duration: 3000, timeoutEvent: 'retry', refreshEvent: 'failed' }
  ],
  callbacks: {}
});
```

Each timeout entry supports 4 properties:

| Property         | Required     | Description                                               |
| -------------    | ------------ | --------------------------------------------------------- |
| **state**        | yes          | The state that will trigger this timeout                  |
| **duration**     | yes          | The timeout length in milliseconds                        |
| **timeoutEvent** | yes          | The stateMachine event to fire when the timeout completes |
| **refreshEvent** | no           | An event that will reset the timeout if not yet completed |


# Storage Helpers

Wilson component scopes are decorated with special storage helper methods for setting and getting localStorage data that
is specific to the component itself. This data is stored to a component-namespaced key that isolates the values stored
from other components. These values should be explicitly set and cleared from the component scope.

## <a name="storageGet"></a>$scope.storage.get(key, defaultValue)

Retrieve a value from this component's localStorage namespace at a given key. If the value does not exist
return the specified defaultValue, otherwise return null.

```typescript
function get(key: string, defaultValue: any): any;
```
Usage Example:
```js
wilson.component('dashboard', {
  controller: ['$scope', function($scope) {
    
    $scope.viewMode = $scope.storage.get('viewMode', 'list-view');
    
    wilson.log.info($scope.viewMode);                         // --> "list-view"
    
    $scope.storage.set('viewMode', 'grid-view');              // --> sets 'viewMode' to local storage
    
    $scope.viewMode = $scope.storage.get('viewMode', 'list-view');
    
    wilson.log.info($scope.viewMode);                         // --> "grid-view"
    
  }]
});
```

## <a name="storageSet"></a>$scope.storage.set(key, value)

Store a value to this component's localStorage namespace at a given key. This will persist in the browser
indefinitely and will be retained when the browser is refreshed or even closed.

```typescript
function set(key: string, value: any): any;
```
Usage Example:
```js
wilson.component('dashboard', {
  controller: ['$scope', function($scope) {
    
    $scope.storage.set('viewMode', 'grid-view');
    
    $scope.viewMode = $scope.storage.get('viewMode', 'list-view');
    
    wilson.log.info($scope.viewMode);   // --> "grid-view"
    
  }]
});
```


# Event Helpers

The component controller interface contains a special set of event handlers under the **auto** sub-property.
These methods provide the same functionality as the original handling methods, but are auto cleaned up when
the scope of the component is destroy. This allows developers to use them freely without having to worry about
memory/reference management.

## <a name="onEvent"></a>$scope.on.event(eventName, handler)

Alias for $scope.$on. See [angularjs documentation](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on).

```typescript
function event(eventName: string, handler: Function): void;
```

## <a name="onWatch"></a>$scope.on.watch(key, handler)

Alias for $scope.$watch. See [angularjs documentation](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$watch).

```typescript
function watch(key: string, handler: Function): void;
```

## <a name="onSignal"></a>$scope.on.signal(signal, handler)

Alias for signal.add. See [js-signal documentation](http://millermedeiros.github.io/js-signals/docs/symbols/Signal.html#add).

```typescript
function signal(signal: Signal, handler: Function): void;
```

> EXPLAINED: This method attaches the **handler** to the **signal** by calling the signal.add.

## <a name="onDigest"></a>$scope.on.digest(handler)

Run the given handler after each digest cycle completes.

```typescript
function digest(handler: Function): void;
```

## <a name="onPageUnload"></a>$scope.on.pageUnload(handler, includeLocalNav)

Run the given handler when the pages is about to unload. If **includeLocalNav** is true, then also
run on local page navigation.

```typescript
function pageUnload(handler: Function, includeLocalNav: boolean): void;
```

# Advanced Component Features

Wilson provides a few additional features on its components that allow for advanced use cases. These features are designed to 
give flexibility to developers when determining the best way to structure their code and optimize loading times.

## The expose attribute

Each component supports a special html attribute called "expose". This attribute can be applied to a component when it is
used in a template and must be supplied a property name as the attribute value. When applied, the sub-component will 
decorate any **exports** onto it's parent component $scope.

When using the expose attribute an **exports** object should be included in the component definition. This object represents
an explicit map of properties that will be "exposed" on the parent component scope.

Example:

**dashboard.html**
```html
<div class="dashboard">
  <ht-news-feed expose="newsFeed"></ht-news-feed>     <!-- Assume "ht" is our component prefix -->
</div>
```
**dashboard.js**
```js
wilson.component('dashboard', {
  controller: ['$scope', '$timeout', function($scope, $timeout) {
    
    wilson.log.info($scope.newsFeed);             // --> This will be null at construction time (since parents load prior to children)
    
    $timeout(function() {
      wilson.log.info($scope.newsFeed.feed);      // --> [{ id: '1234', title: 'Stocks up 20%' }]  - referenced from news-feed's $scope
      wilson.log.info($scope.newsFeed.trending);  // --> undefined (this was not included in the exports object))
    });
    
  }]
});
```
**news-feed.js**
```js
wilson.component('news-feed', {
  exports: {
    feed: 'feed'
  },
  controller: ['$scope', function($scope) {
    
    $scope.feed     = [{ id: '1234', title: 'Stocks up 20%' }];
    $scope.trending = [{ id: '5678', title: 'Some Political BS' }]; 
    
  }]
});
```

In our example above, the news-feed component decorates an object of exported properties onto dashboard as **newsFeed**. This allows the
dashboard component to access the $scope of its child at runtime (post construction). 

The expose feature is very useful when building complex interaction models between components. It adds another dimension to
how developers can structure their code while still decoupling concerns. Out-of-the-box angularjs architecture supports child to parent 
$scope access, but not the other way around. Wilson adds this functionality knowing that there are times when this may be
necessary to achieve a desirable solution.

> NOTE: Legacy support for the expose attribute decorates the entire child $scope onto the parent component $scope. This
> functionality is now deprecated as of wilson v4. Please use the **exports** object to itemize exposed functionality
> as this is more explicit and therefore considered best practice.

## Inheriting from a parent component

In some cases it may be necessary for a child component to inherit some methods directly from its parent component. In most cases
this can be accomplished by using standard isolate scope bindings ([see angularjs scope bindings](https://docs.angularjs.org/api/ng/service/$compile#-scope-)).
However, there are times when it may be more convenient for a child component to directly inherit one or more methods from its parent vs passing them through as
markup attributes. It is for these cases that Wilson provides an extended "inherit" option on component declarations.
 
Example:

**news-feed.html**
```html
<div class="news-feed">
  <ht-feed-item ng-repeat="item in feed" item="item"></ht-feed-item>     <!-- Assume "ht" is our component prefix -->
</div>
```
**news-feed.js**
```js
wilson.component('news-feed', {
  controller: ['$scope', function($scope) {
    
    $scope.feed     = [{ id: '1234', title: 'Stocks up 20%' }, { id: '5678', title: 'Some Political BS' }];
    
    $scope.removeFeedItem = function removeFeedItem(id) {
      _.remove($scope.feed, { id: id });
    };
    
  }]
});
```
**feed-item.html**
```html
<div class="feed-item">
  <h2>[[item.title]]</h2>     <!-- Assume "ht" is our component prefix -->
  <button ng-click="removeItem(item.id)">Remove</button>
</div>
```
**feed-item.js**
```js
wilson.component('dashboard', {
  scope: {
    item: '=item'
  },
  inherit: {
    removeFeedItem: 'removeItem'
  },
  controller: ['$scope', '$timeout', function($scope, $timeout) {
    
    // NOTE:  There are no explicit methods declared here
    
  }]
});
```

In the example above, the parent news-feed component instantiates a list of feed-item components, binding down the feed item using the
"item" attribute. The feed-item component declaration specifies an "inherit" block that states that this component should inherit the
"removeFeedItem" method from its parent onto its own scope with an alias of "removeItem". From the feed-item markup, the "removeItem"
method is called directly using the ng-click directive (as the alias method is automatically applied to the $scope -- no explicit declaration in the controller).


## Dependency hooks

Wilson components provide special hooks to optimize the loading of external scriptDependencies. Many times
a component may need to use a special javascript library to provide its functionality. However, we may not want to add this library to
the core application javascript and increase our load time...Enter wilson dependency hooks.

Wilson allows the developer to specify scriptDependencies in a component definition and will fire hooks to the controller
when those dependencies are ready.

Take for example a dashboard component that needs to use **d3.js**. We only want the d3 library to load when we need to 
instantiate the dashboard component.

Example:

**dashboard.js**
```js
wilson.component('dashboard', {
  scriptDependencies: [
    'https://cdnjs.cloudflare.com/ajax/libs/d3/4.8.0/d3.min.js'
  ],
  controller: ['$scope', '$window', function($scope, $window) {
    
    $scope.showCharts = false;
    
    $scope.onDependenciesReady = function() {       // Fires when dependency scripts are loaded and ready
      $scope.showCharts = true;
    };
    
    $scope.onDependenciesError = function() {       // Fires if dependency scripts fail to load
     $window.alert('Sorry we don\'t have what we need to load');
    };
    
  }]
});
```
If scriptDependencies are defined and non-empty, wilson will attempt to load the scripts when the component is instantiated. The 
$scope may then implement the **onDependenciesReady** and **onDependenciesError** methods in order to handle the
hooks fired by wilson.

> NOTE: Template content that requires the use of scriptDependencies should be manually prevented from rendering prior
> to the **onDependenciesReady** hook. In the example above, $scope.showCharts is meant to represent a flag that controls
> the d3-dependent markup using ng-if="showCharts".



