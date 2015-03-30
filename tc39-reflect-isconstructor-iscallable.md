Reflect.isCallable / Reflect.isConstructable
============================================

##Why are these useful?

- Help support classes/other new function definitions in legacy framework code without significant changes
- Expose a pretty important part of the runtime to applications, who also may wish to use them
- Not require depending on slow (and inconsistent across implementations) `Function.toString()` processing or try/catch statements

## The very tiny normative language:

###Reflect.isCallable(argument)

When the isCallable function is called with argument, the following steps are taken:

1. Return [IsCallable](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-iscallable)(argument)

###Reflect.isConstructor(argument)

When the isConstructor function is called with argument, the following steps are taken:

1. Return [IsConstructor](https://people.mozilla.org/~jorendorff/es6-draft.html#sec-isconstructor)(argument)
