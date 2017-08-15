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

[See the code in ComponentFactoryService](https://github.com/hightail/wilson/blob/4.0.0/lib/client/src/services/ComponentFactoryService.js)

# $scope decorations

* [component](#component)
* [parentComponent](#parentComponent)
* [stateMachine](#stateMachine)
* [defaultValue](#defaultValue)
* [triggerDigest](#triggerDigest)
* [bindToDigest](#bindToDigest)


## <a name="component"></a>$scope.component

An object representing the identification metadata of the component.

```typescript
component: object;
```
Properties:
```json
{
  "id": ""
}
```

Usage Example:
```js
wilson.component('nav-bar', {
  controller: ['$scope', function($scope) {
    var message = 'This is the ' + $scope.componentCName + ' component';
    
    wilson.log.info(message);   // --> "This is the nav-bar component"
  }]
});
```

## <a name="parentComponent"></a>$scope.parentComponent

The scope of this component's parent wilson component (aka it's containing component). This gives components access
(if necessary) to the scope of their parent. If this is a page-level component parentComponent will be null.

> IMPORTANT: **parentComponent** is not necessarily the direct parent $scope, but the $scope of the parent wilson component.

```typescript
parentComponent: Object;
```
Example Parent:

**todo-list.html**
```html
<div class="todo-list">
  <ul>
    <li ng-repeat="todo in todoList">
      <ht-todo todo="todo"></ht-todo>     <!-- Assume  "ht" is our component prefix -->
    </li>
  </ul>
</div>
```
**todo-list.js**
```js
wilson.component('todo-list', {
  controller: ['$scope', function($scope) {
    var controller = this;
    
    $scope.todoList = [{ name: 'Go to the store' }, { name: 'Make Dinner' }, { name: 'Walk the dog'}];
  }]
});
```
Example Child:

**todo.html**
```html
<div class="todo">
  <h3>[[name]]</h3>
  <button ng-click="markComplete()">mark complete</button>
</div>
```
**todo.js**
```js
wilson.component('todo', {
  scope: {
    todo: '=todo'
  },
  controller: ['$scope', function($scope) {
    var controller = this;
    
    $scope.markComplete = function markComplete() {
      $scope.todo.complete = true;
    };
    
    wilson.log.info($scope.parentComponent);            // --> This is the todo-list component $scope
    wilson.log.info($scope.parentComponent.todoList);   // --> [{ name: 'Go to the store }, ...]
  }]
});
```


## <a name="stateMachine"></a>$scope.stateMachine

A special stateMachine object that can be used to add state-based control to a component. Uses the 
[javascript-state-machine library](https://github.com/jakesgordon/javascript-state-machine). 

```typescript
stateMachine: Object;
```

See details below for the [setState() method](#setState).


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

# Controller decorations

* [translate](#translate)
* [overrideText](#overrideText)
* [getPersistentValue](#getPersistentValue)
* [setPersistentValue](#setPersistentValue)
* [setPersistentValues](#setPersistentValues)
* [watchAndPersist](#watchAndPersist)
* [setState](#setState)
* [auto.on](#autoOn)
* [auto.add](#autoAdd)
* [auto.watch](#autoWatch)
* [auto.afterDigest](#autoAfterDigest)



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
    var controller = this;
    
    // Translate to Spanish
    var translatedText  = controller.translate('Where is my dog?', { lng: 'es' });
    
    wilson.log.info(translatedText);    // --> "Donde esta mi perro?"
  }]
});
```

## <a name="overrideText"></a>controller.overrideText(overrideNamespace, textKey, overrideText)

Add an override translation mapping for a component namespace. Used for detailed customization of i18n strings
in a specific context.

```typescript
function overrideText(overrideNamespace: string, textKey: string, overrideText: string): void;
```
Usage Example:
```js
wilson.component('my-component', {
  controller: ['$scope', function($scope) {
    var controller = this;
    
    controller.overrideText($scope.componentCName, 'Where is my dog?', 'Donde esta mi gato?')
    
    // Translate to Spanish
    var translatedText  = $scope.translate('Where is my dog?', { lng: 'es' });
    
    wilson.log.info(translatedText);    // --> "Donde esta mi gato?"
  }]
});
```

## <a name="getPersistentValue"></a>controller.getPersistentValue(key, defaultValue)

Retrieve a value from this component's localStorage namespace at a given key. If the value does not exist
return the specified defaultValue, otherwise return null.

```typescript
function getPersistentValue(key: string, defaultValue: any): any;
```
Usage Example:
```js
wilson.component('dashboard', {
  controller: ['$scope', function($scope) {
    var controller = this;
    
    $scope.viewMode = controller.getPersistentValue('viewMode', 'list-view');
    
    wilson.log.info($scope.viewMode);                         // --> "list-view"
    
    controller.setPersistentValue('viewMode', 'grid-view');   // --> sets 'viewMode' to local storage
    
    $scope.viewMode = controller.getPersistentValue('viewMode', 'list-view');
    
    wilson.log.info($scope.viewMode);                         // --> "grid-view"
  }]
});
```

## <a name="setPersistentValue"></a>controller.setPersistentValue(key, value)

Store a value to this component's localStorage namespace at a given key. This will persist in the browser
indefinitely and will be retained when the browser is refreshed or even closed.

```typescript
function setPersistenValue(key: string, value: any): any;
```
Usage Example:
```js
wilson.component('dashboard', {
  controller: ['$scope', function($scope) {
    var controller = this;
    
    controller.setPersistentValue('viewMode', 'grid-view');
    
    $scope.viewMode = controller.getPersistentValue('viewMode', 'list-view');
    
    wilson.log.info($scope.viewMode);   // --> "grid-view"
  }]
});
```

## <a name="setPersistentValues"></a>controller.setPersistentValues(obj)

Store a set of key/values to this component's localStorage namespace. Returns an object that
contains all stored key/values for this component's namespace. This will persist in the browser
indefinitely and will be retained when the browser is refreshed or even closed.

```typescript
function setPersistentValues(obj: Object): Object;
```
Usage Example:
```js
wilson.component('dashboard', {
  controller: ['$scope', function($scope) {
    var controller = this;
    
    controller.setPersistentValues({ viewMode: 'grid-view', welcomeMessage: 'Hello World' });
    
    $scope.viewMode       = controller.getPersistentValue('viewMode');
    $scope.welcomeMessage = controller.getPersistentValue('welcomeMessage');
    
    wilson.log.info($scope.viewMode);         // --> "grid-view"
    wilson.log.info($scope.welcomeMessage);   // --> "Hello World"
  }]
});
```

## <a name="watchAndPersist"></a>controller.watchAndPersist(key, defaultValue)

Setup a $scope.$watch on the given key that stores the value to this component's localStorage namespace
when the value changes. Stores the defaultValue if the watched value becomes falsey.

```typescript
function watchAndPersist(key: string, defaultValue: any): void;
```
Usage Example:
```js
wilson.component('dashboard', {
  controller: ['$scope', '$timeout', function($scope, $timeout) {
    var controller = this;
    
    $scope.message = 'Hello';
    
    controller.watchAndPersist('message', 'Foo');   // Start the watch
    
    $scope.updateMessage = function updateMessage(message) {
      $scope.message = message;
    };
    
    $scope.updateMessage('World');    // Update the message - this will trigger the watch
    
    $timeout(function() {
      var storedMessage = controller.getPersistentValue('message');
      
      wilson.log.info(storedMessage);   // --> "World"  - The message will be set in localStorage
    })
  }]
});
```

## <a name="setState"></a>controller.setState(config, callback)

Setup a bindable stateMachine on the component $scope. Uses the [javascript-state-machine library](https://github.com/jakesgordon/javascript-state-machine) to create
a state-based control concept for a component. This method takes a specific configuration object unique to the 
[StateMachine.create()](https://github.com/jakesgordon/javascript-state-machine#usage) method of javascript-state-machine.

```typescript
function setState(config: Object, callback: Function): void;
```
Usage Example:

```js
wilson.component('dashboard', {
  controller: ['$scope', '$timeout', function($scope, $timeout) {
    var controller = this;
    
    $scope.message = 'Please Wait...';
    
    // Setup the stateMachine
    controller.setState({
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
      $scope.stateMachine.loaded();
      
      wilson.log.info($scope.stateMachine.current); // --> "Ready"
      wilson.log.info($scope.message);              // --> "Dashboard Ready"
    });
     
    wilson.log.info($scope.stateMachine.current);   // --> "Loading"
    wilson.log.info($scope.message);                // --> "Please Wait..."
  }]
});
```
Wilson supports an extended set of options for state machines. Specifically the **timeouts** array shown in the
example above. The **timeouts** option allows for an array of time-based state transitions that are automatically
controlled by the stateMachine object itself. 

Example stateMachine definition:
```js
controller.setState({
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
})
```

Each timeout entry supports 4 properties:

| Property         | Required     | Description                                               |
| -------------    | ------------ | --------------------------------------------------------- |
| **state**        | yes          | The state that will trigger this timeout                  |
| **duration**     | yes          | The timeout length in milliseconds                        |
| **timeoutEvent** | yes          | The stateMachine event to fire when the timeout completes |
| **refreshEvent** | no           | An event that will reset the timeout if not yet completed |


# Controller event handlers

The component controller interface contains a special set of event handlers under the **auto** sub-property.
These methods provide the same functionality as the original handling methods, but are auto cleaned up when
the scope of the component is destroy. This allows developers to use them freely without having to worry about
memory/reference management.

## <a name="autoOn"></a>controller.auto.on(eventName, handler)

Alias for $scope.$on. See [angularjs documentation](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$on).

```typescript
function on(eventName: string, handler: Function): void;
```

## <a name="autoWatch"></a>controller.auto.watch(key, handler)

Alias for $scope.$watch. See [angularjs documentation](https://docs.angularjs.org/api/ng/type/$rootScope.Scope#$watch).

```typescript
function watch(key: string, handler: Function): void;
```

## <a name="autoAdd"></a>controller.auto.add(signal, handler)

Alias for signal.add. See [js-signal documentation](http://millermedeiros.github.io/js-signals/docs/symbols/Signal.html#add).

```typescript
function add(signal: Signal, handler: Function): void;
```

> EXPLAINED: This method attaches the **handler** to the **signal** by calling the signal.add.

## <a name="autoAfterDigest"></a>controller.auto.afterDigest(handler)

Run the given handler after each digest cycle completes.

```typescript
function afterDigest(handler: Function): void;
```


# Advanced Component Features

Wilson provides a few additional features on its components that allow for advanced use cases. These features are designed to 
give flexibility to developers when determining the best way to structure their code and optimize loading times.

## The expose attribute

Each component supports a special html attribute called "expose". This attribute can be applied to a component when it is
used in a template and must be supplied a property name as the attribute value. When applied, the sub-component will 
decorate its $scope onto the parent component $scope at the specified property.

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
    var controller = this;
    
    wilson.log.info($scope.newsFeed);           // --> This will be null at construction time (since parents load prior to children)
    
    $timeout(function() {
      wilson.log.info($scope.newsFeed.feed);    // --> [{ id: '1234', title: 'Stocks up 20%' }]  - referenced from news-feed's $scope
    });
  }]
});
```
**news-feed.js**
```js
wilson.component('news-feed', {
  controller: ['$scope', function($scope) {
    var controller = this;
    
    $scope.feed = [{ id: '1234', title: 'Stocks up 20%' }];
  }]
});
```

In our example above, the news-feed component decorates its $scope onto dashboard as **newsFeed**. This allows the
dashboard component to access the $scope of its child at runtime. 

The expose feature is very useful when building complex interaction models between components. It adds another dimension to
how developers can structure their code while still decoupling concerns. Out-of-the-box angularjs architecture supports child to parent 
$scope access, but not the other way around. Wilson adds this functionality knowing that there are times when this may be
necessary to achieve a desirable solution.

## Dependency hooks

Component controllers provide special hooks to optimize the loading of external scriptDependencies. Many times
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
    var controller = this;
    
    $scope.showCharts = false;
    
    controller.onDependenciesReady = function() {       // Fires when dependency scripts are loaded and ready
      $scope.showCharts = true;
    };
    
    controller.onDependenciesError = function() {       // Fires if dependency scripts fail to load
     $window.alert('Sorry we don\'t have what we need to load');
    };
    
  }]
});
```
If scriptDependencies are defined and non-empty, wilson will attempt to load the scripts when the component is instantiated. The 
controller may then implement the **onDependenciesReady** and **onDependenciesError** methods in order to handle the
hooks fired by wilson.

> NOTE: Template content that requires the use of scriptDependencies should be manually prevented from rendering prior
> to the **onDependenciesReady** hook. In the example above, $scope.showCharts is meant to represent a flag that controls
> the d3-dependent markup using ng-if="showCharts".



