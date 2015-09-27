Reflect.isCallable / Reflect.isConstructable
============================================

##Why are these useful?

- Help support classes/other new function definitions in legacy framework code without significant changes
- Expose a pretty important part of the runtime to applications, who also may wish to use them
- Not require depending on slow (and inconsistent across implementations) `Function.toString()` processing or try/catch statements

## The very tiny normative language:

###Reflect.isCallable(argument)

When the isCallable function is called with argument, the following steps are taken:

1. ReturnIfAbrupt(argument).
2. If Type(argument) is not Object, return false.
3. If argument does not have a [[Call]] internal method, return false.
4. If argument has a [[FunctionKind]] internal slot with value `classConstructor`, return false.
5. Return true.

(Should be Return "[IsCallable](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-iscallable)(argument)", but adjusted to not report Class constructors as callable, as they throw unconditionally without invoking any author'd code, in the current draft)

###Reflect.isConstructor(argument)

When the isConstructor function is called with argument, the following steps are taken:

1. Return [IsConstructor](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-isconstructor)(argument)
