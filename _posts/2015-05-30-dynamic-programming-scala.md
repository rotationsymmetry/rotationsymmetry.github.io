---
layout: post
title: Dynamic Programming in Scala
comments: true
categories: Scala
---

[Dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming) is a powerful technique to solve a wide variety of problems, ranging from evaluating [fibonacci sequence](https://en.wikipedia.org/wiki/Fibonacci_number) to finding the [shortest path](https://en.wikipedia.org/wiki/Bellman%E2%80%93Ford_algorithm) between two nodes. The mathematical representation of dynamic programing makes it particularly interesting to implement it in a functional programing language like Scala.

<!-- more -->

## The Math
Let's begin with the idea behind dynamic programing. A problem \\(q\\) belongs to a class of problem \\(Q\\), and our goal is to find the solution \\[s=f(q)\\] to solve the problem $q$. Usually it is very difficult or expensive to evaluate $f(q)$ directly. Fortunately the problem class $Q$ admits a short cut: if we know the solutions to some problems $p_1$, $p_2$, ...,  $p_k$ (the choice of $p$'s depends on $q$), there is a simple process $H$ to find the solution to $q$ by combining the solutions to the $p$'s, e.g. \\[f(q)=H(f(p_1),f(p_2),f(p_k))\\] This shortcut will allow us to solve the problems recursively. We just need to make sure that $p$'s will eventually reduce to the simple problem with solution readily available and the recursion will indeed terminate for our initial problem $q$.

## Example: Splitting 10 Cents
If you have many 1-cent and 5-cent coins, how many ways to split 10 cents? We can define a problem class $Q$ as: how to many ways to split $n$ with $c_1$, $c_2$, ... $c_k$. The scala code provided below along with comments to explains what it is going.

``` scala
// Define the problem set: split n with a list of coin denominations
// The denominations should be unique.
case class Question(n: Int, coins: List[Int])

def splitCoins(q: Question): Int = {
  println(s"split ${q.n} with ${q.coins}")
  q.coins match {
    case h::t => {
      // Reduction: Three possibilities depending on n and the first denomination
      if (q.n > h)
        splitCoins(Question(q.n-h, q.coins)) + splitCoins(Question(q.n, t))
      else if (q.n == h)
        1 + splitCoins(Question(q.n, t))
      else
        0 + splitCoins(Question(q.n, t))
    }
    // Boundary problem: no coin denomination can be used => zero way to split n
    case _ => 0
  }
}
```

Please note in the reduction steps, either $n$ or the number of available denominations decreases. So the recursive will eventually hit the boundary problems and it will always terminate.

## Referential Transparency
Simply put, [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency_(computer_science)) means that the value of an expression remains the same regardless the context/scope in which the expression is evaluated. As one of the hallmarks of functional programming, referential transparency makes a program much easier to reason with: you always have a definitive idea of what an expression means, no matter where you encounter it.

Back to the coin problem, the number of ways to split 10 cents with 1-cent and 5-cents will always be same way, regardless you are requested do it in a pub or in a supermarket. The same applies to the general formulation of the dynamic programming too. The value of $f(q)$ is invariant of when and where you evaluate it. Because of this, in your code, you only need to describe the reduction process and boundary problems without worrying or managing the evaluation of the expression. This will enhances readability of your code and productivity of your team.

## Speed it Up with Scalaz
Looking at the `println` messages, you will notice that some splitting problems (such as `split 3 with List(1)`) have been solved multiple times. Because the solution to a problem will always be the same, it is redundant to repeat the evaluation many times. Instead, we can evaluate the solution once and then cache the value for future use. We are guarantee this approach will give us the correct result because of referential transparency.

In many other languages, caching is managed manually with storage structure such as a hash map. In Scala, we can automate caching with [Scalaz](https://github.com/scalaz/scalaz), a library that further enhances the functional programing capability of Scala.

Implement the caching with Scalaz cannot be easier. The only modification is to supply the function body of the non-caching split coin algorithm to `Memo.mutableHashMapMemo`, which will return a new function with caching using a `HashMap`. Not a single bit of boilerplate code to manage the cache get into the way. Very clean solution indeed.

``` scala
val memoSplitCoins: Question => Int = Memo.mutableHashMapMemo {q => {
  println(s"split ${q.n} with ${q.coins}")
  q.coins match {
    case h::t => {
      if (q.n > h)
        splitCoins(Question(q.n-h, h::t)) + splitCoins(Question(q.n, t))
      else if (q.n == h)
        1 + splitCoins(Question(q.n, t))
      else
        0 + splitCoins(Question(q.n, t))
    }
    case _ => 0
  }
}}
```

When you run the `memoSplitCoin`,  split coin problem will never be evaluated more than once according to the `println` messages.

## Immutable Data Structure
