---
layout: post
title: DSL and the Craft of Do-Until Loop
comments: true
---
Scala has a `while` loop built-in. But can we craft a `Do-until` loop? It is fun to review Scala features that make it possible and debate how we should implement DSL.

<!-- more -->

Talk is cheap. Show you the the  code

``` scala
class Do(block: => Unit) {
    def until (cond: => Boolean): Unit = {
        block
        while (!cond) {
            block
        }
    }
}

object Do {
    def apply(block: => Unit) = new Do(block)
}
```

Let's test it out in the REPL and it works as expected. 

``` scala
scala> var x = 0
x: Int = 0

scala> Do {
     |     println(s"x = $x")
     |     x = x + 1
     | } until (x == 3)
x = 0
x = 1
x = 2

scala>
```

The implementation of `Do-until` loop takes advantage of two Scala features. 

### Call by Name Parameter
`block : => Unit` is a *call by name* parameter in the constructor of the `Do` class. A call by name parameter will not be evaluated when it is defined; instead it is evaluated every time it is called. I agree that it is a bit abstract. Personally I prefer to consider call by name parameter as a function with no argument:

``` scala
def f1(x1: => Int) = {
	x1
	x1
}

def f2(x2: ()=>Int)  = {
	x2()
	x2()
}

// test it in REPL
scala> f1({println("evaluate x1"); 3})                                                                                                                                                                
evaluate x1
evaluate x1
res0: Int = 3

scala> f2(()=>{println("evaluate x2"); 3})
evaluate x2
evaluate x2
res1: Int = 3
```

`evaluate x1` and `evaluate x2` are printed twice because `x1` and `x2` are called twice in the function `f1` and `f2`, respectively. So `f1` and `f2` are semantically equivalent. One more example of syntax sugar at work!

### Infix Notation
Essentially, you can a method of an instance without the `.` and the parenthesis:

``` scala 
foo bar 1 // is the same as foo.bar(1)
```  

So the infix notation allows the `until` method of the `Do`  class to appear as if it is a native keyword from the language. 


### My Take on DSL
With the `Do-until` example, you can see that Scala is extremely expressive and you are free to create new syntax as you please. So it is very common in the Scala community to develop new syntax dubbed as Domain Specific Language (DSL). 

However, you can does not always mean you should. With the help of any modern IDE, saving keystrokes is never a legitimate reason to create a new syntax. Also, for every line of the code you wrote in 10 seconds, you, your colleagues and other contributors are likely to spend hours of time reading them repeatedly. So it turns out to be much more productive to stick to the common denominator for everyone's knowledge and familiarity of the language. 

Simplicity and uniformity are cherished more than complexity and fragmentations. I would keep it in mind when creating new DSL.
