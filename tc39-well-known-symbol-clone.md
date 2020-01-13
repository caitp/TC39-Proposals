@@clone
=======

## Rationale

It is useful to construct copies of objects, for use in dirty checking (tracking a new state vs old state of an object), or for creating an immutable/hidden clone. An abstract convention for cloning objects is useful as it allows simplifying the code needed for generalized cloning operations.

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

1. **Cloning host objects**

	Despite creating the clone object using the host object's prototype, a native C++ instance of the host object is not created.

	In some engines, accessing any properties of this object specified in IDL will result in an exception.

2. **Cloning collections introduced in ES2015**

	Data in collections is not stored as properties of the collection, and as such, copying properties does not accomplish the requested task. It takes a fair bit of special casing code in order to handle cloning these objects.

	Weak collections can't be cloned as engines have rejected providing any API that would require iteration, such as the `clear()` method.

## @@clone conventions:

The `@@clone` function is a method whose `this` value is the object being cloned, and which receives a parameter indicating
whether the clone should be a "deep" clone (whereby nested objects should be similarly cloned). As such, a shallow clone
would create copies of object values, while a deep clone would create new objects.

## Reflect.clone ( <var>O</var> [, <var>deep</var>] )

When the `clone` function is called with arguments <var>O</var> and <var>deep</var>, the following steps are taken:

1. If [Type](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)(<var>O</var>) is not Object, then return <var>O</var>.
1. Let <var>deep</var> be ! [ToBoolean](https://tc39.es/ecma262/#sec-toboolean)(<var>deep</var>).
1. Let <var>cloneFn</var> be ? [GetMethod](https://tc39.es/ecma262/#sec-getmethod)(<var>obj</var>, @@clone).
1. If <var>cloneFn</var> is **undefined**, then set <var>cloneFn</var> to [%Object.prototype[@@clone]%](#objectprototype--clone---deep-).
1. Else if [IsCallable](https://tc39.es/ecma262/#sec-iscallable)(<var>cloneFn</var>) is **false**, then throw a **TypeError** exception.
1. Let <var>result</var> be ? [Call](https://tc39.es/ecma262/#sec-call)(<var>cloneFn</var>, <var>obj</var>, « <var>deep</var> »).
1. If [Type](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)(<var>result</var>) is not Object, then throw a **TypeError** exception.
1. Return <var>result</var>.

## Set.prototype \[ @@clone ] ( <var>deep</var> )

1. Let <var>O</var> be ? [ToObject](https://tc39.es/ecma262/#sec-toobject)(**this** value).
1. Let <var>deep</var> be ! [ToBoolean](https://tc39.es/ecma262/#sec-toboolean)(<var>deep</var>).
1. Let <var>ctor</var> be ? [SpeciesConstructor](https://tc39.es/ecma262/#sec-speciesconstructor)(<var>O</var>, [%Set%](https://tc39.es/ecma262/#sec-set-constructor))
1. Let <var>result</var> be ? [Construct](https://tc39.es/ecma262/#sec-construct)(<var>ctor</var>).
1. Let <var>iterator</var> be ? [GetIterator](https://tc39.es/ecma262/#sec-getiterator)(<var>O</var>).
1. Repeat,
	1. Let <var>next</var> be ? [IteratorStep](https://tc39.es/ecma262/#sec-iteratorstep)(<var>iterator</var>).
	1. If <var>next</var> is **false**, then return <var>result</var>.
	1. Let <var>nextValue</var> be ? [IteratorValue](https://tc39.es/ecma262/#sec-iteratorvalue)(<var>next</var>).
	1. If [Type](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)(<var>nextValue</var>) is Object, then
		1. If <var>deep</var> is **true**, then set <var>nextValue</var> to ? [Call](https://tc39.es/ecma262/#sec-call)([%Reflect.clone%](#reflectclone-o-deep-), undefined, « <var>nextValue</var>, **true** »).
	1. Let <var>result</var> be ? [Invoke](https://tc39.es/ecma262/#sec-invoke)(<var>result</var>, **"add"**, « <var>nextValue</var> »).

## Map.prototype \[ @@clone ] ( <var>deep</var> )

1. Let <var>O</var> be ? [ToObject](https://tc39.es/ecma262/#sec-toobject)(**this** value).
1. Let <var>deep</var> be ! [ToBoolean](https://tc39.es/ecma262/#sec-toboolean)(<var>deep</var>).
1. Let <var>ctor</var> be ? [SpeciesConstructor](https://tc39.es/ecma262/#sec-speciesconstructor)(<var>O</var>, [%Map%](https://tc39.es/ecma262/#sec-map-constructor))
1. Let <var>result</var> be ? [Construct](https://tc39.es/ecma262/#sec-construct)(<var>ctor</var>).
1. Let <var>iterator</var> be ? [GetIterator](https://tc39.es/ecma262/#sec-getiterator)(<var>O</var>).
1. Repeat,
	1. Let <var>next</var> be ? [IteratorStep](https://tc39.es/ecma262/#sec-iteratorstep)(<var>iterator</var>).
	1. If <var>next</var> is **false**, then return <var>result</var>.
	1. Let <var>nextValue</var> be ? [IteratorValue](https://tc39.es/ecma262/#sec-iteratorvalue)(<var>next</var>).
	1. Let <var>nextKey</var> be Get(<var>nextValue</var>, 0).
	1. Set <var>nextValue</var> to Get(<var>nextValue</var>, 1).
	1. If [Type](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)(<var>nextValue</var>) is Object, then
		1. If <var>deep</var> is **true**, then set <var>nextValue</var> to ? [Call](https://tc39.es/ecma262/#sec-call)([%Reflect.clone%](#reflectclone-o-deep-), undefined, « <var>nextValue</var>, **true** »).
	1. Let <var>result</var> be ? [Invoke](https://tc39.es/ecma262/#sec-invoke)(<var>result</var>, **"set"**, « <var>nextKey</var>, <var>nextValue</var> »).

## Object.prototype \[ @@clone ] ( <var>deep</var> )

1. Let <var>O</var> be ? [RequireObjectCoercible](https://tc39.es/ecma262/#sec-requireobjectcoercible)(**this** value).
1. If [Type](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)(<var>O</var>) is not Object, then return <var>O</var>.
1. Let <var>deep</var> be ! [ToBoolean](https://tc39.es/ecma262/#sec-toboolean)(<var>deep</var>).
1. Let <var>proto</var> be ? <var>O</var>.\[\[GetPrototypeOf]]().
1. Let <var>result</var> be [ObjectCreate](https://tc39.es/ecma262/#sec-objectcreate)(<var>proto</var>).
1. Let <var>keys</var> be ? <var>O</var>.\[\[OwnPropertyKeys]]().
1. For each element <var>key</var> of <var>keys</var>, do
	1. Let <var>desc</var> be ? <var>O</var>.\[\[GetOwnProperty]](<var>key</var>).
	1. If [Type](https://tc39.es/ecma262/#sec-ecmascript-data-types-and-values)(<var>desc</var>.\[\[Value]]) is Object, then
		1. If <var>deep</var> is **true**, then set <var>desc</var>.\[\[Value]] to ? [Call](https://tc39.es/ecma262/#sec-call)([%Reflect.clone%](#reflectclone-o-deep-), undefined, « <var>nextValue</var>, **true** »).
	1. ! [DefinePropertyOrThrow](https://tc39.es/ecma262/#sec-definepropertyorthrow)(<var>result</var>, <var>key</var>, <var>desc</var>).

## Similar Proposals:

1. [Structured cloning and transfer](https://github.com/dslomov-chromium/ecmascript-structured-clone)
  Advantages: Deals with cloning between realms --- Disadvantages: limited to host objects and builtins
