# Scalameta tutorial: Cache decorator

This is a tutorial to show how to use Scalameta to develop a **generic, parameterized annotation**.
To know how to setup a project to use `scalameta`, refer to [official docs](http://scalameta.org/tutorial/#Setup)

## What is scalameta?

[Scala-meta](http://scalameta.org/) is the de-facto toolkit for metaprogramming in Scala. For those who are new to metaprogramming, it means programming against code/syntax/[AST](https://en.wikipedia.org/wiki/Abstract_syntax_tree)

Metaprogramming is very useful when you notice a repeating pattern in your code, but you are not able to refactor it due to limitations of the programming language.

Conceptually, scalameta allows you to access your code as data (Abstract Syntax Tree), and manipulate it at compile time.

## Problem: Caching

Caching is a common technique that almost all programmers are familiar with. In this tutorial, we will develop a `cache` macro that

* has low syntatic overhead, ie. it should not change the cached function much
* able to support different cache storage, eg. In memory cache, elastic search etc
* able to cache methods with multiple arguments

## Naive implementation

Let's start with a simple implementation without using macro

Code to support cache function
```scala
class CacheBackEnd[K, V] {
  private var map = mutable.Map[K, V]     // ignore the fact it is not thread safe

  // `compute` is a function, only evaluated in case of cache miss
  def getOrElse(k: K, compute: K => V): V = {
    map.get(k) match {
      case Some(v) => v       // cache hit
      case None =>
        val v = compute(k)
        map = map + (k -> v)
        v
    }
  }
}

def cache[K, V](fn: K => V)(cacheStorage: CacheBackEnd[K, V]): K => V = {
  (k: K) =>
    cacheStorage.getOrElse(k, fn)
}
```

Calling cache function
```scala

// this is slow .....
def fib(i: Int): Int = i match {
  case 0 => 0
  case 1 => 1
  case n => fib(n - 2) + fib(n - 1)
}

val cacheBackend = new CacheBackEnd[Int, Int]

def cachedFib(i: Int): Int = cache(fib)(cacheBackend)(i)

// alternatively, we can inline the logic
def cachedFib(i: Int): Int = cache {
  case 0 => 0
  case 1 => 1
  case n => fib(n - 2) + fib(n - 1)
}(cacheBackend)(i)
```

### Analysis
**Pros**
- it's simple and easy to understand
- it supports different cache storage
- able to work on methods of different arguments

**Cons**
- It is a bit intrusive, notice the implementation of `cachedFib` needs to be changed
- It is awkward to work with functions with multiple arguments, because our CacheBackend can only takes in `K, V` as type param, so if you have a function with signature `def fn(x: Int, y: Int): Int` , you need to combine `x` and `y` into `(Int, Int)` so that it fit into CacheBackend's type signature

Let's see how we can improve it using scala meta.

## Scalameta Implementation 1

Here, we are going to implement cache function as a macro, the end goal is to support syntax like this

```scala

// the signature is slightly different, as our macro need to access `get` and `put` method, but not `getOrElse`
// you can implement both signature though
trait SyncCache[K, V] {
  def get(k: K): Option[V]
  def put(k: K, v: V): Unit
}

val cacheBackend = new SyncCache[Int, Int]

@cache(cacheBackend)
def fib(i: Int): Int = i match {
  case 0 => 0
  case 1 => 1
  case n => fib(n - 2) + fib(n - 1)
}

```

Before we jump into implementation, we can observe a few difference with previous implementation

### Observations
- The syntax is cleaner, it does not change the method's definition at all
- It also supports different cache storage
- Too much MAGIC, how does it even work?
- Does it support function with multiple arguments?

Let's answer the 1st question, how does it works?

Below is the implementation of the `cache` macro, let's go through the comments to understand what it does

```scala
// @param backend - parameter for `cache` annotation
class cache[K, V](backend: SyncCache[K, V]) extends scala.annotation.StaticAnnotation {

  // @param defn - the annotated method (it can also be other scala building block like class, but we are restricting here
  //               using some checks below
  inline def apply(defn: Any): Any = meta {

    defn match {
      // this annotation should only be annotate on `method`, represented as `Defn.Def` in scalameta's AST
      case defn: Defn.Def =>

        // this represent the instatiated annotation, ie. `@cache(xxx)`
        this match {

          // Quasiquote in action, it's a way to pattern match scala AST
          //  - `new`               match instantiation
          //  - `$_`                match the name of the annotation
          //  - `[..$tpr]`          match type params and bind to `tpr`
          //  - `($backendParam)`   match SINGLE argument used in instatiation and bind to `backendParam`
          //                        Note: To match multiple arguments or multiple arguments list (curried), you need different                                           syntax
          case q"new $_[..$tpr]($backendParam)" =>

            // we use the element we captured to generate code we want, we will look into details next
            val body: Term = CacheMacroImpl.expand(tpr, backendParam, defn)

            // we only want to replace the method body, so this will do
            defn.copy(body = body)
          case x =>
            abort(s"Unrecognized pattern $x")
        }

      case _ =>
        abort("This annotation only works on `def`")    // abort if annotated to anything other than methods
    }
  }
}
```

So here is quite some amount of info, especially around quasiquote. You might have a few questions, like what is the type signature of `tpr` that we've captured? I will go through them in next section, but here I wish you get familiar with the general flow, basically we are trying to

1. use pattern matching to capture relevant information from the AST [Compile Time]
2. perform transformation on the AST [Compile Time]
3. transformed AST will then get compiled into artifact that is invoked at runtime

if you're interested to know more about quasiquote, [here](https://github.com/scalameta/scalameta/blob/master/notes/quasiquotes.md) is the reference for all quasiquote syntax

Now let's inspect the implementation of AST transformation logic, ie. `CacheMacroImpl.expand(tpr, backendParam, defn)`

```scala
object CacheMacroImpl {

  /**
    *
    * @param fnTypeParams - Type params of annotation instance, remember our cache macro is generic `class cache[K, V]`,
                            this will capture Seq(K, V)
    * @param cacheExpr    - Argument pass to `cache` macro, should be type of `CacheBackEnd[K, V]`
    * @param annotatedDef - Methods that is annotated
    */
  def expand(fnTypeParams: Seq[Type], cacheExpr: Term.Arg, annotatedDef: Defn.Def): Term = {

    val cache = Term.Name(cacheExpr.syntax)     // convert Term.Arg to Term.Name

    annotatedDef match {
      // Another Quasiquote pattern match
      // - `..$_`                   match any modifier
      // - `def`                    match only method
      // - `$methodName`            bind method name to $methodName
      // - `[..$tps]`               match some type params of method and bind to $tps
      // - `(..$nonCurriedParams)`  match non-curried argument list and binf to $nonCurriedParams
      // - `$rtType`                bind return type to $rtType
      // - `$expr`                  bind method's body to $expr
      case q"..$_ def $methodName[..$tps](..$nonCurriedParams): $rtType = $expr" =>

        // here is the trick to handle different arg size
        if (nonCurriedParams.size == 1) {

          // if only 1 arg, use the arg as key of cache
          val paramAsArg = Term.Name(nonCurriedParams.head.name.value)
          q"""
            // here we are generating code that call the CacheBackend
            val result: ${rtType} = $cache.get($paramAsArg) match {
              case Some(v) => v
              case None =>
                val value = ${expr}
                $cache.put($paramAsArg, value)
                value
            }
            result
           """
        } else {
          val paramAsArg = nonCurriedParams.map(p => Term.Name(p.name.value))
          q"""

            // if there are multiple arg, convert them in tuple, as use the tuple as key
            val result: ${rtType} = $cache.get((..$paramAsArg)) match {
              case Some(v) => v
              case None =>
                val value = ${expr}
                $cache.put((..$paramAsArg), value)
                value
            }
            result
           """
        }
      case other => abort(s"Expected non-curried method, got $other")
    }
  }
}
```

I hope the implementation is not too intimidating, it does the following

1. Check if the annotation methods is allowed or not (curried method is not allowed)
2. Check the number of arguments
    - If 1, use it as is
    - If multiple, convert them into a tuple, and use the tuple as key of cache
3. Try to get data from cache using `cache.get(key)`
    - If cache hit, return the cached value
    - If cache miss, evaluate the original annotated method, cache the result using `cache.put(k, v)`, and return the result

### Analysis
**Pros**
- it supports different cache storage, eg. you could implement a Cache Backend that support TTL
- it has almost zero syntatic overhead on caller
- able to work on methods of different arguments

**Cons**
- It is more complicated to implement
- The implementation is harder to debug

## Conclusion
Here we end this tutorial, as we shown how could you create a generic, parameterize macro using scalameta.
The code is availble [here](https://github.com/buaya91/scala-cache)
As an exercise for readers, you can try to improve the cache so that it support async get and put.

Note: I am not claiming `cache` annotation is a good use-case of macro, ultimately it depends on your team and problem on hand. Nonetheless, I believe everyone should learn a bit of it to enhance your skills, and also to have better understanding on how compiler view your code.
