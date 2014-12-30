@@clone
=======

##Rationale

It is useful to construct copies of objects, for use in dirty checking (tracking a new state vs old state of an object), or for
creating an immutable clone.

In ES5, naive strategies for doing this are used:

```js
function Clone(O) {
  var clone = Object.create(Object.getPrototypeOf(O));
  Object.getOwnPropertyNames(O).forEach(function(key) {
    clone[key] = O[key];
  });
  return clone;
}
```

Unfortunately, this strategy is problematic in the following cases:

1. Cloning host objects
  Despite creating the clone object using the host object's prototype, a native C++ instance of the host object is not created.
  In some engines, accessing any properties of this object specified in IDL will result in an exception.
2. Cloning ES6 collections
  Data in collections is not stored as properties of the collection, and as such, copying properties does not accomplish the
  requested task. It takes a fair bit of special casing code in order to handle cloning these objects.

## @@clone conventions:

The `@@clone` function is a method whose `this` value is the object being cloned, and which receives a parameter indicating
whether the clone should be a "deep" clone (whereby nested objects should be similarly cloned). As such, a shallow clone
would create copies of object values, while a deep clone would create new objects.

## Reflect.clone(O, deep):

When the `clone` function is called with arguments `O` and `deep`, the following steps are taken:

1. If `Type(O)` is not `Object`, then return `O`.
2. If `O` is `null`, then return `O`.
3. Let `deep` be `ToBoolean(deep)`.
4. `ReturnIfAbrupt(deep)`.
5. Let `cloneFn` be `GetMethod(obj, @@clone)`.
6. `ReturnIfAbrupt(cloneFn)`.
7. If `cloneFn` is not `undefined`, then
    1. Let `result` be `Call(cloneFn, obj, deep)`.
    2. `ReturnIfAbrupt(result)`.
    3. If `Type(result)` is not `Object`, then throw a `TypeError` exception.
    4. Return `result`.
8. else throw a `TypeError` exception.

The `length` of `Reflect.clone` is 1.

## Set.prototype \[@@clone](deep):

1. Let `result` be a new `Set`.
2. Let `iterator` be `GetIterator(this)`.
3. ReturnIfAbrupt(iterator).
4. Let `deep` be `ToBoolean(deep)`.
5. Repeat
  1. Let `next` be `IteratorStep(iterator)`.
  2. `ReturnIfAbrupt(next)`.
  3. If next is `false`, then return `result`.
  4. Let nextValue be `IteratorValue(next)`.
  5. `ReturnIfAbrupt(nextValue)`.
  6. Let `nextValue` be `Get(nextValue, 0)`.
  7. If `Type(nextValue)` is `Object`, then
    1. If `deep` is `true`, then let `nextValue` be `Reflect.clone(nextValue, true)`.
  8. Let `result` be `result.add(nextValue)`.
  9. `ReturnIfAbrupt(result)`.

## Map.prototype \[@@clone](deep):

1. Let `result` be a new `Map`.
2. Let `iterator` be `GetIterator(this)`.
3. ReturnIfAbrupt(iterator).
4. Let `deep` be `ToBoolean(deep)`.
5. Repeat
  1. Let `next` be `IteratorStep(iterator)`.
  2. `ReturnIfAbrupt(next)`.
  3. If next is `false`, then return `result`.
  4. Let `nextValue` be `IteratorValue(next)`.
  5. `ReturnIfAbrupt(nextValue)`.
  6. Let `nextKey` be `Get(nextValue, 0)`.
  7. Let `nextValue` be `Get(nextValue, 1)`.
  8. If `Type(nextValue)` is `Object`, then
    1. If `deep` is `true`, then let `nextValue` be `Reflect.clone(nextValue, true)`.
  9. Let `result` be `result.set(nextKey, nextValue)`.
  10. `ReturnIfAbrupt(result)`.

## Object.prototype\[@@clone](deep):

1. Let `O` be `ToObject(this)`.
2. `ReturnIfAbrupt(O)`.
3. Let `proto` be the result of calling the `[[GetPrototypeOf]]` internal method of `O`.
4. `ReturnIfAbrupt(proto)`.
5. Let `result` be `ObjectCreate(proto)`.
6. Let `keys` be the result of calling the `[[OwnPropertyKeys]]` internal method of `O`.
7. `ReturnIfAbrupt(keys)`.
8. Let `iterator` be `GetIterator(keys)`.
9. Repeat
  1. Let `next` be `IteratorStep(iterator)`.
  2. `ReturnIfAbrupt(next)`.
  3. If next is `false`, then return `result`.
  4. Let `nextValue` be `IteratorValue(next)`.
  5. `ReturnIfAbrupt(nextValue)`.
  6. Let `nextKey` be `Get(nextValue, 0)`.
  7. Let `nextValue` be `Get(nextValue, 1)`.
  8. If `Type(nextValue)` is `Object`, then
    1. If `deep` is `true`, then let `nextValue` be `Reflect.clone(nextValue, true)`.
  9. Let `putStatus` be `Put(result, nextKey, nextValue, true)`.
  10. `ReturnIfAbrupt(putStatus)`.

## Similar Proposals:

1. [Structured cloning and transfer](https://github.com/dslomov-chromium/ecmascript-structured-clone)
  Advantages: Deals with cloning between realms --- Disadvantages: limited to host objects and builtins
