# Wilson Components

Components are the heart of a wilson application. They are custom elements that contain a subset
of functionality that can be re-used as needed. Components can represent a page (the target 
of a route) or a building-block that is used to as a distinct piece of a user interface. 

Under-the-hood, a component is just an angularjs "element-style" directive that has isolateScope and defines
an explicit controller [see angularjs components](https://docs.angularjs.org/guide/component). But what makes
wilson components any different than just using angularjs directly?

Good question.

From the server, wilson provides dependency resolution for all components in the codebase. This means
that the wilson server reads and determines all dependencies between components/behaviors/services in
the application. This makes it possible to lazy-load components as they are needed when the app
navigates to a new area. No registration or importing is required by the developer, simply create
a component and start using it...Magic.

Apart from dependency resolution, wilson also applies specialized base functionality onto each component. This
ensures that each component has the same base interface and gives the developer access to
common functions and utilities, which helps to eliminate boiler plate and code duplication. These 
special decorations are split between the $scope and the controller instance of a component.  

[See how it works in ComponentFactoryService](https://github.com/hightail/wilson/blob/3.0.0/lib/client/src/services/ComponentFactoryService.js)

# Component $scope decorations

* [componentCName](#componentCName)
* [parentComponent](#parentComponent)
* [stateMachine](#stateMachine)
* [defaultValue](#defaultValue)
* [triggerDigest](#triggerDigest)
* [bindToDigest](#bindToDigest)


# Component controller decorations

* [translate](#translate)
* [overrideText](#overrideText)
* [setState](#setState)
* [getPersistentValue](#getPersistentValue)
* [setPersistentValue](#setPersistentValue)
* [setPersistentValues](#setPersistentValues)
* [watchAndPersist](#watchAndPersist)
* [auto.on](#autoOn)
* [auto.add](#autoAdd)
* [auto.watch](#autoWatch)
* [auto.afterDigest](#autoAfterDigest)



## <a name="translate"></a>translate(text, options)

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

## <a name="overrideText"></a>overrideText(overrideNamespace, textKey, overrideText)

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

# Advanced Component Features

Wilson provides a few additional features on its components that allow for advanced component
interactions. These features are designed to give flexibility to developers when determining the best
way to structure their code.

## The Expose Attribute

