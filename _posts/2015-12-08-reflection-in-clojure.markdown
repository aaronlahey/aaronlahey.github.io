---
layout: post
title: "Reflection in Clojure"
date: 2015-12-08
categories: clojure
---

It's always been a personal goal of mine to write a unit testing framework. I think it presents some interesting problems, teaches you more about the language you're writing it in, and is software that could potentially be useful in everyday development. I also want to start diving deeper into certain topics. I've spent many hours improving the breadth of my knowledge, but not nearly as much the depth in a single topic. Writing not just a testing a framework, but a test framework people would actually *want to use* is proving to be an excellent start to that journey.

As one can imagine, writing a framework which allows you to write code, execute that code, and provides meaningful output describing the verification of that code involves a handful of knowledge about how to reflect on the code itself. Important things to know are how to define functions, add them to namespaces, search all namespaces, get the file certain functions are located in, the line number they're defined on, etc. Fortunately, Clojure has *a ton* of useful built-in utilities for reflecting on code; some of which I hope to cover in this post.

## Namespaces

In this post I'd like to focus on namespaces. There are a few helpful functions and variables built in to Clojure for reflecting on the namespaces within a clojure project. The first is the global dynamic variable `*ns*` which is always bound to the current namespace...

{% highlight clojure %}
user=> *ns*
#<clojure.lang.Namespace@22784b32 user>
{% endhighlight %}

Next we have `all-ns` which, as one would expect, lists all namespaces currently loaded in the project...

{% highlight clojure %}
user=> (all-ns)
(#<clojure.lang.Namespace@22784b32 coconut.util>
 #<clojure.lang.Namespace@629eeaf4 coconut.reporting>
 #<clojure.lang.Namespace@5a0ba6df clojure.tools.nrepl.ack>
 #<clojure.lang.Namespace@3328a6fe clojure.tools.nrepl.middleware.render-values>
 #<clojure.lang.Namespace@53e95a72 coconut.test>
 #<clojure.lang.Namespace@46b5c3b1 clojure.stacktrace>
 #<clojure.lang.Namespace@410fa8e6 clojure.core.rrb-vector.transients>
 #<clojure.lang.Namespace@3766f96f coconut.leiningen-test>
 #<clojure.lang.Namespace@2ec51f3b clojure.tools.namespace.track>
 #<clojure.lang.Namespace@4685f919 clojure.uuid>
 #<clojure.lang.Namespace@7a7131b7 complete.core>)
{% endhighlight %}

The namespaces returned from `*ns*` and `all-ns` are the actual `clojure.lang.Namespace` objects as opposed to the symbols used to represent their names, such as `coconut.util` or `coconut.reporting`. This is an important distinction if you're trying to test for equality using the values returned from them...

{% highlight clojure %}
user=> (= 'user *ns*)
false
{% endhighlight %}

To retrieve the symbol name of a namespace, Clojure ships with a helpful function called `ns-name`. This function can be used in combination with anything that returns namespace objects to test for equality...

{% highlight clojure %}
user=> (= 'user (ns-name *ns*))
true

user=> (-> (into (hash-set)
  #_=>           (map ns-name)
  #_=>           (all-ns))
  #_=>     (contains? 'coconut.util))
true
{% endhighlight %}

Clojure namespaces hold references to the vars defined within them. We can use a function called `ns-publics` to return a map from symbols to the vars to which they will ultimately resolve...

{% highlight clojure %}
user=> (defn foo
  #_=>   []
  #_=>   (println "hello world"))
#'user/foo

user=> (ns-publics *ns*)
{apropos-better #'user/apropos-better,
 cdoc #'user/cdoc,
 clojuredocs #'user/clojuredocs,
 find-name #'user/find-name,
 foo #'user/foo,
 help #'user/help,
 run-all-tests #'user/run-all-tests,
 run-all-tests* #'user/run-all-tests*,
 run-current-namespace #'user/run-current-namespace,
 run-current-namespace* #'user/run-current-namespace*,
 user.proxy$java.lang.Object$SignalHandler$d8c00ec7 #'user/user.proxy$java.lang.Object$SignalHandler$d8c00ec7}
{% endhighlight %}

Even with just these few functions we have the ability to start treating our vars and namespaces as data. Just composing these functions together and using them with Clojure's collection operations allows us to create new functions; functions we can use to help us learn more about this topic!

{% highlight clojure %}
user=> (defn ns-functions
  #_=>   []
  #_=>   (into (vector)
  #_=>         (comp (mapcat (comp keys ns-publics))
  #_=>               (filter (comp (partial re-matches #"ns\-.*") str)))
  #_=>         (all-ns)))

user=> (ns-functions)
[ns-java-methods
 ns-public-vars
 ns-vars
 ns-classes
 ns-functions
 ns-unmap
 ns-publics
 ns-unalias
 ns-aliases
 ns-resolve
 ns-refers
 ns-name
 ns-map
 ns-interns
 ns-imports]
{% endhighlight %}

Hopefully this function we created gives you a starting point to learn more about Clojure namespace functions. However, we've only scratched the surface. In future posts I hope to dive deeper into namespaces as well discuss vars and metadata. Thanks for reading!