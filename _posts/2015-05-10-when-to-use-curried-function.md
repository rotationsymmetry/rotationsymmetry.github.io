---
layout: post
title: When to Use Curried Function
comments: true
tags: scala
---
A friend of mine recently brought up the question of when to define a function in the curried style. Frankly speaking, I seldom use currying in my own projects. Sematically there is no real difference between a curried function and its uncurried counterpart. However, when we can consider the currying in the context of Scala, I believe there are a few scenarios that currying is worth considering.

<!-- more -->

## Implicit Parameters
I have discussed implicit parameters in another [post](/2015/05/13/implicit-parameter/). In the definition of a function, the parameters within the pair parenthesis forms a parameter group. The `implicit` keyword act on the parameter group instead of individual parameter.

``` scala
scala> def f(x: Int)(implicit y: Int, z: Double) = x + y + z
f: (x: Int)(implicit y: Int, implicit z: Double)Double
```

Therefore, you need to separate non-implicit and implicit parameters into different parameter groups, which naturally introduces the currying notation.

One more side note, the `implicit` keyword can only be placed in the right most parameter group. Otherwise it is an error.

``` scala
scala> def f(implicit x: Int)(y: Int) = x + y
<console>:1: error: '=' expected but '(' found.
       def f(implicit x: Int)(y: Int) = x + y
                             ^
```

## Type Inference
Scala supports type inference, which saves you a lot of keystrokes compared to Java. However, the inference algorithm still has some limitations. Specifically, the inferred type information cannot be shared among the parameters within the same pair of parenthesis. This sounds quite abstract so let's illustrate it using `List[T]` as an example.

``` scala
case object Empty extends List[Nothing]
case class Con[T](head: T, tail: List[T]) extends List[T]
sealed class List[+T] {

  def foldLeft[S](s: S, f: (S, T) => S): S = this match {
    case Empty => s
    case Con(head, tail) => tail.foldLeft(f(s, head), f)
  }
}
```

Pretty standard isn't it? Let's test `foldLeft`:

``` scala
scala> val data = Con(1, Con(2, Con(3, Empty)))

scala> data.foldLeft(0, (x, y)=>x+y)
Error:(20, 17) missing parameter type
data.foldLeft(0, (x, y)=>x+y)
               ^
```

Why does Scala complains that missing parameter type for `x`? It is true that the type inference algorithm can figure out the type `S`, for `0`, is `Int`. However, the knowledge for `S` does not apply to type inference of the anonymous function because it is within the same pair parenthesis of `0`. As a result, Scala can't figure out the type of `x` as well as the signature of `f`. One solution to this issue is to annotate the type of `x` in the anonymous function.

``` scala
scala> data.foldLeft(0, (x :Int, y)=>x+y)
res0: Int = 6
```

While this works, it is quite tedious and error prone to do the annotation everywhere. A better solution is to use currying. The trick is that the type inference algorithm can broadcast the type information *across* parenthesis from left to right. In `curryFoldLeft` below, once the type `S` is determined from the pair of parethesis on the left, the type inference algorithm will also know the signature of `f`.

``` scala
  def curryFoldLeft[S](s: S)(f: (S, T) => S): S = this match {
    case Empty => s
    case Con(head, tail) => tail.curryFoldLeft(f(s, head))(f)
  }
```

Let's test `curryFoldLeft` without annotating the signature of the anomymous function. It works as expected:

``` scala
scala> data.curryFoldLeft(0)((x, y)=>x+y)
res0: Int = 6
```

Taking advantage of the characteristics of Scala type inference algorithm, we can define function in the curried style to minimize signature annotation. It is no surprise that the default `foldLeft` of `List[T]` in the Scala Library use currying.
