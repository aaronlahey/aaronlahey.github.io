---
layout: post
title: "Functions, Monoids, and Monads"
date: 2016-06-05
categories: fp
---

> A monad is a monoid in the category of endofunctors

There's a good chance you've seen this quote or some variation of it floating around the internet. You may have even heard some of your colleagues say it. Typically it's said in jest; an impenetrable, somewhat sarcastic response to the question: "What is a monad?" In this blog post I'd like to break this sentence down, explain its components, and show you that it actually is a nice, succinct explanation of monads.

## Sets

The dictionary definition for a set is this: "A number, group, or combination of things of similar nature, design, or function." The idea of a set of things in everyday programming is not particularily complex and can be represented as a particular type. For example, if we have a value with a type of `String`, the type system is telling is "This value can be any member from the set of all possible `String`s." Of course, the set of all possible `String`s is infintely large, but that isn't always the case. It's possible to have a type represent a finite set. For example:

{% highlight scala %}
sealed trait CMYK
case object Cyan extends CMYK
case object Magenta extends CMYK
case object Yellow extends CMYK
case object Key extends CMYK

{% endhighlight %}

If we declared a value as type `CMYK` it could only be one of four possible values: `Cyan`, `Magenta`, `Yellow`, or `Key`. A function which outputs the names of these colors as a string would have type `CMYK => String`, meaning it is a direct mapping from the finite set of `CMYK`s to the infinite set of all possible `String`s.

## Monoids

We can build on our knowledge of sets to understand monoids. A monoid is formed when there exists a set of things, an operation which combines two members of that set, and there is some notion of a zero value, which we can call `unit`. The value of `unit` depends on the operation we define, but we can define it abstractly by saying it is any value within our given set which satisfies the left/right identity laws. These laws state that if we perform `x operation unit` the result should be `x`. If we perform `unit operation x` the result should also be `x`. This is a bit abstract, so let's concretize it a bit.

The first example will be simple integer addition. Our set will be the set of all possible `Int`s and our operation will be `+`. The identity element for integer addition is obvious: `0`. However, we *know* it happens to be the identity element because it satisfies the left/right identity laws: `n + 0 = n` and `0 + n = n`.

We can define other monoids for the set of all `Int`s; multiplication being another example. Even though we're using the same set of values, we're using a different operation and, consequently, a different identity value. Again, even though `1` is the obvious identity element we can prove it with the following examples: `n * 1 = n` and `1 * n = n`.

For a set and an operation to form a monoid one other law must be satisfied: associativity. This means we should be able to group the combination of multiple elements in any way and it has no effect on the outcome. Integer addition, for example, is associative because `1 + (2 + 3) = 6` and `(1 + 2) + 3 = 6`. Integer multiplication is also associated: `(1 * 2) * 3 = 6` and `1 * (2 * 3) = 6`.

For a more real-world example of a monoid we can look at validations. Suppose we have a `User` object which has a `name` and an `age`. We'll perform a validation on the `name` which checks that it is not blank and a validation on the `age` which checks that it is greater than zero. Once we've performed these individual validations we'll want to combine them to see if the `User` as a whole is valid and if not, have an aggregated set of all the error messages.

{% highlight scala %}
case class User(name: String, age: Int)
case class ValidationResult(valid: Boolean, errors: Set[String]) {
  def ++(other: ValidationResult): ValidationResult {
    ValidationResult(valid && other.valid, errors ++ other.errors)
  }
}

def validateName(user: User): ValidationResult = user.name match {
  case "" => ValidationResult(false, Set("Required"))
  case _ => ValidationResult(true, Set())
}

def validateAge(user: User): ValidationResult = user.age match {
  case n if n <= 0 => ValidationResult(false, Set("Must be greater than 0"))
  case _ => ValidationResult(true, Set())
}

val user = User("", 0)
validateName(user) ++ validateAge(user) //=> ValidationResult(false, Set("Required", "Must be greater than 0"))

{% endhighlight %}

In this example our set of values is the set of all possible `ValidationResult`s, our operation for combining two `ValidationResult`s is the `++` operator, and even though it isn't utilized here there exists a `unit`:

{% highlight scala %}
val unit = ValidationResult(true, Set())

unit ++ ValidationResult(false, Set("Required", "Must be greater than 0"))
  //=> ValidationResult(false, Set("Required", "Must be greater than 0"))
 
ValidationResult(false, Set("Required", "Must be greater than 0")) ++ unit
  //=> ValidationResult(false, Set("Required", "Must be greater than 0"))
  
{% endhighlight %}