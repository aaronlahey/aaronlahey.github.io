---
layout: post
title: "The Relationship Between Functions, Symbols, Vars, and Namespaces"
authors: ["aaron-lahey"]
date: 2016-07-20
tags: ["Coding", "Tools"]
---

Welcome! Often times when working with Clojure newcomers I sense a bit of confusion around the relationship between functions, symbols, vars, and namespaces. Typically the developer has a working knowledge of how to require other namespaces or refer functions, but they haven't quite grasped how all the pieces fit together allowing them to execute the expressions they're writing. Also, although we typically refer to functions using the names of the symbols to which they're aliased, I get the feeling beginners do it out of a general lack of understanding of these topics instead of as a sort of shorthand in everyday conversation. In this blog post I'd like to explore what exactly these four things are, how they work together, some useful functions for interacting with them, and how we can use our newfound knowledge to build tools that reflect on or modify the state of the runtime.

## Functions

If you're writing code in Clojure you probably have a good sense of what a function is. However, you might not know how Clojure represents functions internally. Knowing exactly what a Clojure function *is*, is vital to explaining what a function *isn't*.

Internally, the language represents functions as any Java class that implements the <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/IFn.java" target="_blank">`clojure.lang.IFn`</a> interface. If we look at <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/IFn.java" target="_blank">the source</a> for `IFn` we see it contains two methods that must be implemented: `invoke`, which contains 22 overloads, and `applyTo`. This interface gives us some insight into how the language implements different function arities: one method for each arity up to 20, and one utilizing Java varargs for arities greater than 20. Implementing 23 methods just to create a new function would not only be a pain, but the majority of the required function arities probably wouldn't even apply to your use case. Also, your implementation of `applyTo` would generally be the same across all functions.

To help alleviate this problem, Clojure ships with the abstract base class <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/AFn.java" target="_blank">`clojure.lang.AFn`</a>. This class provides a default implementation of `applyTo` and implements all `invoke` overloads to throw a <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/ArityException.java" target="_blank">`clojure.lang.ArityException`</a>. In fact, you've probably seen this default implementation at work and didn't even realize it!

```Clojure
(ns example.core)

(defn add
  [x y]
  (+ x y))

(add 1 2 3) ;;=> ArityException Wrong number of args (3) passed to: example.core/add  clojure.lang.AFn.throwArity (AFn.java:429)
```

Now that we know a function is a class that implements `clojure.lang.IFn`, we can begin to implement our own functions in Java. For example, here's a Java implementation of an addition function which, unlike `clojure.core/+`, only supports adding two `long`s together.

```java
package example.core;

import clojure.lang.AFn;

public class Add extends AFn {
  @Override
  public Object invoke(Object x, Object y) {
    return ((long) x) + ((long) y);
  }
}
```

Once this class has been imported, we can invoke instances of it like we would any other Clojure function...

```clojure
(import '[example.core Add])

((Add.) 1 2) ;;=> 3
((Add.) 1 2 3) ;;=> ArityException Wrong number of args (3) passed to: Add  clojure.lang.AFn.throwArity (AFn.java:429)

(apply (Add.) [1 2]) ;;=> 3
(apply (Add.) [1 2 3]) ;;=> ArityException Wrong number of args (3) passed to: Add  clojure.lang.AFn.throwArity (AFn.java:429)
```

Notice when we invoked an instance of the `Add` class there was no mention of any symbolic names. We didn't refer to it as `+` or `add`, we just instantiated an object and invoked it. So, if the built-in Clojure addition function is just an instance of a class without any association to an identifier, what exactly *is* `+`?

## Symbols

Symbols in Clojure are just pieces of data. From an implementation perspective they're instances of <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/Symbol.java" target="_blank">`clojure.lang.Symbol`</a>. They can be passed as function arguments or put into a collection just like a keyword or a string. Clojure uses them as identifiers for various values and references, which gives them a bit of a special status in the language, but if we take care to quote them in our code or at the repl they're usable just like any other value...

```clojure
'foo ;;=> foo
'+ ;;=> +
['foo 'bar 'baz] ;;=> [foo bar baz]
```

### Named

Symbols implement the <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/Named.java" target="_blank">`clojure.lang.Named`</a> interface, which means they have the ability to return a `String` description of their namespace as well as their name using the methods `getNamespace()` and `getName()` respectively. We can also use functions provided by `clojure.core` to inspect these values...<span id="source-1"><a href="#footnote-1"><sup>1</sup></a></span>

```clojure
(namespace 'bar) ;;=> nil
(name 'bar) ;;=> "bar"

(namespace 'foo/bar) ;;=> "foo"
(name 'foo/bar) ;;=> "bar"
```

One important thing to realize about symbols is that they aren't functions<span id="source-2"><a href="#footnote-2"><sup>2</sup></a></span>. When you execute `(+ 1 2)` you're not invoking the symbol `+`, you're invoking the function which the symbol `+` refers to based on the current state of the runtime. At this point you may be thinking that Clojure maintains some internal mapping between symbols and values. However, if Clojure only maintained a *static* mapping from symbols to values or functions (instances of `clojure.lang.IFn`), it would be severely limiting. We wouldn't be able to have more than one function with a given name in our entire program, and we wouldn't be able to dynamically change what a symbol refers to—the former being vital to any programming language, and the latter allowing Clojure to be as dynamic as it is.

## Vars

Vars (instances of <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/Var.java" target="_blank">`clojure.lang.Var`</a>) are one of four constructs the Clojure language gives us to maintain a persistent reference to a changing value.<span id="source-3"><a href="#footnote-3"><sup>3</sup></a></span> In fact, of the four constructs (<a href="http://clojure.org/reference/vars" target="_blank">vars</a>, <a href="http://clojure.org/reference/atoms" target="_blank">atoms</a>, <a href="http://clojure.org/reference/refs" target="_blank">refs</a>, and <a href="http://clojure.org/reference/agents" target="_blank">agents</a>), they're probably the one you use the most without even realizing it! Every time something is defined using the special form `def` (or the `defn` macro which uses `def` internally), Clojure creates a new var and places the value or function inside...

```clojure
(ns example.core)

(def x 42) ;;=> #'example.core/x

(var-get #'x) ;;=> 42

(defn increment
  [x]
  (inc x)) ;;=> #'example.core/increment

(var-get #'increment) ;;=> #<Fn@7a2f7ac example.core/increment>
```

Having this mutable container act as a layer of indirection between a caller and the function being called is what allows Clojure to be so dynamic at runtime. If all invocations of a particular function are routed through this container we're given the opportunity to dynamically update the behavior of our program by changing or altering what is inside...

```clojure
(ns example.core)

(defn increment
  [x]
  (inc x))

(increment 0) ;;=> 1

(alter-var-root #'increment (partial comp increment))

(increment 0) ;;=> 2
```

Notice the reference to *root* in the name of the `alter-var-root` function. A var's reference to a value is referred to as a *binding* and vars have two different kinds: a root binding which is visible to all threads, and a dynamic thread-local binding. When we `def` a value we're specifically setting that value as the *root* binding of the newly-created var. Although altering the root binding of a var has its use cases, the most common way a Clojure developer interacts with vars is by declaring them dynamic and utilizing the `binding` macro to update their thread-local binding within a particular lexical scope...

```clojure
(ns example.core)

(def ^:dynamic x 0) ;;=> #'example.core/x

(println x) ;;=> 0
(binding [x 42]
  (println x)) ;;=> 42
(println x) ;;=> 0
```

The same way that invoking `(+ 1 2)` gives the illusion that `+` is a function, the syntax of the binding macro gives the illusion that `x` is a var which can be rebound within the scope of the `binding` call. In both cases they're just symbols; pieces of data that resolve to something based on the state of the runtime. If we resort to Java interop to interact with vars we can get this same behavior without specifically referring to them using symbolic identifiers...

```clojure
(import '[clojure.lang Var])

;; root binding
(Var/create 42) ;;=> #'Var: --unnamed-->
(var-get (Var/create 42)) ;;=> 42

;; thread-local binding
(let [x (.setDynamic (Var/create 0))]
  (do (println (var-get x)) ;;=> 0
      (with-bindings {x 42}
        (println (var-get x))) ;;=> 42
      (println (var-get x)))) ;;=> 0
```

Notice in this example we needed to add extra ceremony around retrieving the current value of the var. Typically, when a var is created using `def` and we reference the symbol which maps to it, Clojure will find the var and return its current binding, not the var itself. Here, however, since we've assigned an instance of a var to `x` within a `let` binding, Clojure does not automatically return its binding. Sometimes it can be unclear whether the macro you're using requires a symbol and does the resolving internally (such as `binding`) or the actual instance of a var (such as `with-bindings`). Understanding the difference between a symbol and a var and being able to distinguish between the two is vital to clearing up this otherwise surprising behavior.

Now that we know vars hold functions and values, and those vars are referenced using a symbol, how does Clojure maintain and resolve the mapping between the symbol `x` and a specific var?

## Namespaces

Namespaces (instances of <a href="https://github.com/clojure/clojure/blob/master/src/jvm/clojure/lang/Namespace.java" target="_blank">`clojure.lang.Namespace`</a>) are Java objects that provide *mappings* between unqualified symbols and vars, or *aliases* that are mappings between unqualified symbols and other namespaces. In order to inspect a namespace, we need to first get our hands on an instance of one. Clojure comes with a few built-in methods for retrieving a namespace. First, we can always use `*ns*`, which is a dynamic var which will always be bound to the namespace in which we're currently executing...

```clojure
(ns example.core)

*ns* ;;=> #<clojure.lang.Namespace@606bf74f example.core>
```

In this example, we can see the var is currently bound to an instance of a `clojure.lang.Namespace`. This namespace object has a name, which happens to be the unqualified symbol `example.core`. Using `*ns*` to return the current namespace can lead to surprising behavior if you're unsure of when exactly the var will be dereferenced. Consider the following function...

```clojure
(ns example.foo)

(defn namespace-object
  [] *ns*)

(ns example.bar
  (:require [example.foo :as foo]))

(foo/namespace-object) #=> #<clojure.lang.Namespace@5425625d example.bar>
```

The confusion here arises because our `namespace-object` function essentially relies on global mutable state. Once we've entered the `example.bar` namespace the `*ns*` var's binding has changed and no longer reflects what it did when we defined the function. What we wanted all along was to return the value of the dynamic var *at the time it was defined* instead of the time when it was executed...

```clojure
(ns example.foo)

(let [this-namespace *ns*]
  (defn namespace-object
    [] this-namespace))

(ns example.bar
  (:require [example.foo :as foo]))

(foo/namespace-object) #=> #<clojure.lang.Namespace@5425625d example.foo>
```

It's also possible load a namespace by its name using either the `find-ns` or `the-ns` functions. These two functions are similar, but `find-ns` returns `nil` when the namespace is not found, and `the-ns` throws an exception. Also, `the-ns` accepts namespace objects as well as symbols...

```clojure
(ns example.foo)
(ns example.bar)

(find-ns 'example.foo) ;;=> #<clojure.lang.Namespace@1261b1bb example.foo>
(find-ns 'example.bar) ;;=> #<clojure.lang.Namespace@462c18eb example.bar>
(find-ns 'example.baz) ;;=> nil
(find-ns (find-ns 'example.foo)) ;;=> ClassCastException clojure.lang.Namespace cannot be cast to clojure.lang.Symbol  clojure.core/find-ns (core.clj:3996)

(the-ns 'example.foo) ;;=> #<clojure.lang.Namespace@1261b1bb example.foo>
(the-ns 'example.bar) ;;=> #<clojure.lang.Namespace@462c18eb example.bar>
(the-ns 'example.baz) ;;=> Exception No namespace: example.baz found  clojure.core/the-ns (core.clj:4032)
(the-ns (the-ns 'example.foo)) ;;=> #<clojure.lang.Namespace@1261b1bb example.foo>
```

### Mappings and Aliases

If we want to retrieve all aliases in a given namespace, we can use the `ns-aliases` function, which returns a map from unqualified symbols to namespace instances. The resulting mappings will typically reflect namespaces that have been required using the `ns` macro's `:as` syntax...

```clojure
(ns example.foo)
(ns example.bar
  (:require [example.foo :as ef]))

(ns-aliases 'example.foo) ;;=> {}
(ns-aliases 'example.bar) ;;=> {ef #<clojure.lang.Namespace@1af4e9eb example.foo>}
```

If we want to retrieve all symbol to *var* mappings within a particular namespace, we can use the `ns-map` function...

```clojure
(ns example.foo)

(defn add
  [x y]
  (+ x y))

(ns-map 'example.foo) ;;=>

{add #'example.foo/add
 primitives-classnames #'clojure.core/primitives-classnames,
 +' #'clojure.core/+',
 Enum #<Class@6b6776cb java.lang.Enum>,
 decimal? #'clojure.core/decimal?,
 restart-agent #'clojure.core/restart-agent,
 sort-by #'clojure.core/sort-by,
 macroexpand #'clojure.core/macroexpand,
 ensure #'clojure.core/ensure,
 ;; many, many more
 coll? #'clojure.core/coll?,
 get-in #'clojure.core/get-in,
 fnext #'clojure.core/fnext,
 denominator #'clojure.core/denominator,
 bytes #'clojure.core/bytes,
 refer-clojure #'clojure.core/refer-clojure}
```

If you execute this in a repl you'll see there are upwards of 700 mappings in a brand new namespace. Where are all these coming from!? It turns out, the `ns` macro takes the liberty of mapping all public vars from `clojure.core` as well as creating mappings for all of `java.lang`. This is why we don't need to explicitly require Clojure or Java in order to execute `+` or refer to `Exception`. Last we'll look at `ns-publics`, which is similar to `ns-map` except it only returns public vars that are defined directly in the relevant namespace...

```clojure
(ns example.foo)

(defn ^:private multiply
  [x y]
  (* x y))

(defn add
  [x y]
  (+ x y))

(ns-publics 'example.foo) ;;=> {add #'example.foo/add}
```

## Symbol Application

To put the pieces together let's imagine we want to build our own version of `apply`. However, instead of taking a *function* and a collection of arguments, we want it to take a symbol, resolve it to a var, get the current binding of that var, and execute the bound function with the arguments. If it can't resolve the var, we'd like it to simply return `nil`. What would something like that look like? Well, before we write some code, let's describe exactly what we want to happen...

1. If the symbol does not have a namespace description associated to it, we want to look through all mappings (not aliases) in the current namespace. If we don't find a corresponding var we'll just return `nil`. However, if we find a var, we'll get its current binding and apply the arguments to it.<span id="source-4"><a href="#footnote-4"><sup>4</sup></a></span>

2. If the symbol *does* have a namespace description, the most common case would be for it to be a reference to an alias in the current namespace. So, we'll look through all aliases and return the namespace object that corresponds to it. We'll then look through all *public* vars defined within that namespace (not mapped from other namespaces), get the var, get its current binding, and execute the function.

3. If the symbol has a namespace description but for some reason we're unable to locate a corresponding alias in our current namespace, we'll assume it's fully qualified and referring directly to the name of some namespace. If we *are* able to locate that namespace, just as above, we'll look in all public vars defined within it, get the var, get its current binding, and execute the function. If there is no corresponding namespace with that name, we'll just return `nil`.

```clojure
(ns example.core
  (:refer-clojure :exclude [+])
  (:import [example.core Add]))

(def + (Add.))
(def increment (partial + 1))
```

```clojure
(ns example.repl
  (:require [example.core :as core
                          :refer [increment]]))

(defn resolve-symbol
  [s]
  (if-let [namespace-name (namespace s)]
    (when-let [referenced-namespace (or (get (ns-aliases *ns*)
                                             (symbol namespace-name))
                                        (find-ns (symbol namespace-name)))]
      (get (ns-publics referenced-namespace)
           (symbol (name s))))
    (get (ns-map *ns*)
         (symbol (name s)))))

(defn symbol-apply
  [s arguments]
  (when-let [referenced-var (resolve-symbol s)]
    (apply (var-get referenced-var) arguments)))

(symbol-apply 'increment [1]) ;;=> 2
(symbol-apply 'core/+ [1 2]) ;;=> 3
(symbol-apply 'example.core/+ [1 2]) ;;=> 3
```

If I found myself writing this code in a real system I would seriously question my intentions, but I think it's a good example to demonstrate the relationship between these four topics. It shows how we can go from a simple piece of data, through a namespace, through the current binding of a var to some function, and execute it. In fact, this is the path Clojure takes every time you execute an expression or type an unquoted symbol at the repl!<span id="source-5"><a href="#footnote-5"><sup>5</sup></a></span>

I think the internals of Clojure are super interesting, and I hope this post has helped shed some light on the inner workings of symbol resolution and function execution. I also hope you've gained a somewhat deeper knowledge of the lesser-used reflective functions in `clojure.core`. If your interest is piqued and you want to learn more, you can't go wrong by diving into the Clojure <a href="https://github.com/clojure/clojure" target="_blank">source code</a>. Thanks for reading!

---

<span id="footnote-1"><a href="#source-1"><sup>1</sup></a></span> Most public functions in `clojure.core` provide additional behavior and don't just delegate to a corresponding Java method call. For example, while the `namespace` function *does* simply delegate to `getNamespace()`, the `name` function has additional logic to handle Java strings. Execute `(clojure.repl/source namespace)` and `(clojure.repl/source name)` at the repl to see this in action.

<span id="footnote-2"><a href="#source-2"><sup>2</sup></a></span> That's a lie. Symbols do implement `clojure.lang.IFn` and can be invoked. They can be used to look themselves up in a map or set similar to keywords. However, invoking them is not the same as invoking the function to which they resolve.

<span id="footnote-3"><a href="#source-3"><sup>3</sup></a></span> <a href="http://clojure.org/reference/vars" target="_blank">http://clojure.org/reference/vars</a>

<span id="footnote-4"><a href="#source-4"><sup>4</sup></a></span> Getting the current binding of the var in order to execute it as a function isn't strictly necessary. Vars implement `clojure.lang.IFn` and delegate all methods to their current bindings. This is often useful if you need to pass a function to another function while still being able to dynamically change what it does. For example, <a href="http://www.http-kit.org/server.html#stop-server" target="_blank">HTTP Kit</a> suggests doing this if you want to start a web server while still being able to hot-reload code.

<span id="footnote-5"><a href="#source-5"><sup>5</sup></a></span> That's not always true. Check out <a href="http://clojure.org/reference/compilation#directlinking" target="_blank">direct linking</a> for more information.
