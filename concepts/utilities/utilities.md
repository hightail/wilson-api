## Wilson Utilities

Wilson provides a number of utility functions that are decorated onto the global "wilson" instance. These
functions provide various conveniences to developers.

From the code, these methods can be accessed on either the global "wilson" instance or by injecting the "WilsonUtils"
service into any component controller, behavior directive, service or filter declaration.

E.G.

```
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

## Utility Functions

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
* [printStackTrace](#printstacktrace)
* [path.join](#path.join)
* [keyCodes](#keycodes)


### spliceArray

Remove values from an array and optionally replace a set number of new items into the array. This function modifies
the given targetArray, values are removed and replaced on the given array reference. The array is first modified to 
remove items between the start/end indices and then the replacement items are inserted into the
array from the startIndex.

```
spliceArray(targetArray: any[], spliceStartIdx: ?number, spliceEndIdx: ?number, replacements: ?any[])
 

// E.G.
 
var foo          = [1, 2, 3];
var replacements = ['bob', 5];

wilson.utils.spliceArray(foo, 1, 1, replacements);

console.log(foo) ->  [1, 'bob', 5, 3];


```


### replaceArray

Replace the content of the given destinationArray with the items of the sourceArray. The original destinationArray 
reference is preserved, the destinationArray is effectively cleared and then filled with the items of sourceArray.

```
replaceArray(destinationArray: any[], sourceArray: any[])
 
// E.G.
 
var dest    = [1, 2, 3];
var source  = [4, 5, 6];
var ref     = dest;
 
wilson.utils.replaceArray(dest, source);
 
console.log(dest) --> [4, 5, 6]
 
assert(dest === ref); --> true

```


### clearArray

Clear all items from the given array reference (mutating the given targetArray).

```
clearArray(targetArray: any[]);
 
// E.G.
 
var list    = [1, 2, 3];
var ref     = list;
 
wilson.utils.clearArray(list);
 
console.log(list) --> []
 
assert(list === ref); --> true


```