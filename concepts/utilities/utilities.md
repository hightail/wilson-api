# Wilson Utilities

Wilson provides a number of utility functions that are decorated onto the global "wilson" instance. These
functions provide various conveniences to developers.

From the code, these methods can be accessed on either the global "wilson" instance or by injecting the "WilsonUtils"
service into any component controller, behavior directive, service or filter declaration.

E.G.

```js
// Usage from the "wilson" global  - (most convenient)
wilson.utils.clearObject();

 
// Usage from the injectable "WilsonUtils" service
wilson.service('MyService', ['WilsonUtils', function(WilsonUtils) {
  
  var service = {
    doSomethingAndClear: function doSomethingAndClear(obj) {
        // Do something here
        WilsonUtils.clearObject(obj);
    }
  };
  
  return service;
}];

```

# Utility Functions

* [spliceArray](#splicearray)
* [replaceArray](#replacearray)
* [clearArray](#cleararray)
* [replaceObject](#replaceobject)
* [clearObject](#clearobject)
* [getPropFromPath](#getpropfrompath)
* [setPropFromPath](#setpropfrompath)
* [bytesToReadable](#bytestoreadable)
* [generateUUID](#generateuuid)
* [parseBoolean](#parseboolean)
* [bool](#parseboolean)
* [path.join](#path.join)
* [keyCodes](#keycodes)


## spliceArray

Remove values from an array and optionally replace a set number of new items into the array. This function modifies
the given targetArray, values are removed and replaced on the given array reference. The array is first modified to 
remove items between the start/end indices and then the replacement items are inserted into the
array from the startIndex.

```typescript
function spliceArray(targetArray: any[], startIdx?: number, endIdx?: number, replacements?: any[]): any[];
```
Usage Example:
```js
var foo          = [1, 2, 3];
var replacements = ['bob', 5];
 
wilson.utils.spliceArray(foo, 1, 1, replacements);
 
console.log(foo); // -->  [1, 'bob', 5, 3];

```


## replaceArray

Replace the content of the given destinationArray with the items of the sourceArray. The original destinationArray 
reference is preserved, the destinationArray is effectively cleared and then filled with the items of sourceArray.

```typescript
function replaceArray(destination: any[], source: any[]): any[];
```
Usage Example:
```js
var dest    = [1, 2, 3];
var source  = [4, 5, 6];
var ref     = dest;
 
wilson.utils.replaceArray(dest, source);
 
console.log(dest);    // --> [4, 5, 6]
 
assert(dest === ref); // --> true

```


## clearArray

Clear all items from the given array reference (mutating the given targetArray).

```typescript
function clearArray(targetArray: any[]): void;
```
Usage Example:
```js
var list    = [1, 2, 3];
var ref     = list;
 
wilson.utils.clearArray(list);
 
console.log(list);    // --> []
 
assert(list === ref); // --> true

```


## replaceObject

Replace the contents of one object with another. Clears all properties from the destination object and then copies
all new properties from the source object to the destination. The reference to the destination object remains in
tact and is mutated. The source object remains unmodified.

```typescript
function replaceObject(destination: Object, source: Object): void;
```
Usage Example:
```js
var dest    = { foo: 'bar' };
var source  = { hello: 'world' };
var ref     = dest;
 
wilson.utils.replaceObject(dest, source);
 
console.log(dest);    // --> { hello: 'world' }

assert(dest === ref); // --> true

```


## clearObject

Clear all properties of a given object. Mutates passed in the object reference.

```typescript
function clearObject(targetObj: Object): void;
```
Usage Example:
```js
var obj = { foo: 'bar' };
var ref = obj;
 
wilson.utils.clearObject(obj);
 
console.log(obj);     // --> {}

assert(obj === ref);  // --> true
```


## getPropFromPath

Retrieve a property from an object using "property dot" notation. Returns null if not found.

```typescript
function getPropFromPath(obj: Object, path: string): any;
```
Usage Example:
```js
var obj = {
  foo: { 
    bar: 'Hello'
  }
};
 
var bar = wilson.utils.getPropFromPath(obj, 'foo.bar');
 
console.log(bar);   // --> "Hello"

```


## setPropFromPath

Set a property on an object using "property dot" notation. The entire path will be created. If properties in the
path do not exist, they will be provisioned.

```typescript
function setPropFromPath(obj: Object, path: string, value: any): void;
```
Usage Example:
```js
var obj = {};
 
wilson.utils.getPropFromPath(obj, 'foo.bar', 'Hello');
 
console.log(obj.foo.bar);   // --> "Hello"
```


## <a name="bytesToReadable"></a>bytesToReadable(bytes, decimalPoint)

Convert a number representing a byte count to a human readable size in Bytes, KB, MB, GB etc. This method uses a 
base 2 kibi-byte calculation where 1 kilobyte === 1024 bytes. Default decimalPoint is 1.

```typescript
function bytesToReadable(bytes: number, decimalPoint?: number): string;
```
Usage Example:
```js
var rawFileSize   = 107520000;
var readableSize  = wilson.utils.bytesToReadable(rawFileSize);
var preciseSize   = wilson.utils.bytesToReadable(rawFileSize, 3);
 
console.log(readableSize);    // -->  "102.5 MB"
console.log(preciseSize);     // -->  "102.549 MB"
```


## generateUUID()

Creates and returns a RFC4122 v4 compliant UUID string.  Useful for creating identifiers for entities within an application.

```typescript
function generateUUID(): string;
```


## <a name="parseboolean"></a>parseBoolean(value)

Casts a given value to a boolean. This method is more exhaustive than the typical !! casting strategy. Specifically, it will 
identify string values of 'false', '0', 'NaN', 'null', and 'undefined' as falsey.

```typescript
function parseBoolean(value: any): boolean;
```
Usage Example:
```js
var truthyString = 'false';
 
console.log(!!truthyString);                            // --> true
console.log(wilson.utils.parseBoolean(truthyString));   // --> false

```

> NOTE: As of v3.0.0 a shorthand **bool()** method has been added as an alias to **parseBoolean()**



## path.join



## keyCodes