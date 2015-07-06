---
title: Building Web Application with Django
tags: python
published: false
---

Last month I was to develop a prototype web application for cancer research. The application involves structured CRUD ops using simple html forms as well as some relatively complex business logics. I ended up completing this in about one week with python and the Django web framework. Here are my experiences.

<!-- more -->

## Ventured out of the JVM Realm
At the beginning, I was planning to use a Scala-based stack, i.e. [Play](http://www.playframework.com) for web framework, and [Slick](http://slick.typesafe.com/) for database access. The primary reasons are two fold

* The business logics expressed in functional programming languages are more readable.
* The IDE can provide much better hinting and checking for static and strongly typed languages.

However, before I really dig in, I realized that the documentation for Play and Slick were not as clear and comprehensive as I would expect. Also the Slick API were not stable at the time. Incompatible API would be introduced in the soon-to-be-released version 3. As a newbie to Slick, I did not think I could make contributions to the project by being a guinea pig.

I have considered other technology on the JVM platform. They are often either too heavyweight or without good documentations. So I ventured outside of the JVM realm and eventually decided to use Django for my project.

## Why I Chose Django
My decision to choose Django is driven by two factors:

* Documentations
Concise, complete, detailed, well-organized, filled with necessary background information and context. The documentation team of the Django project set the best example for quality documentation. This gives me a tremendous boost in confidence for taking on my own project.

* Django ORM
Super easy to use and understand (again kudo to the documentation). With Django ORM, I can dive into development immediately instead of spending several days configuring the database or thinking how to tie the data table to my models. While the syntax and capability are not perfect, they are good enough for most use cases.

Once the decision is made, I started to get my hands dirty the Python + Django stack.

## Development Experiences
I would say the overall development process are very smooth and fast paced.

I would like to share some stories that are dramatically extreme different from my previous experiences with Scala or Java.


### Running Naked without a Compiler
At its root, python is a dynamic language with duck typing. Your code are interpreted at the run-time directly without the compiler getting in the way. This means you can do a lot of crazy things unimaginable in Scala or Java.

For example, you are working on Feature B and the code are in a complete mess with wrong types, syntax error etc. For whatever reason, you now need to run some tests for Feature A. Then what can you do? In python, you can just go ahead and run your tests for Feature A as long as it does not depend on Feature B. In the Scala or Java, you will need to spend some time setting up all the scaffolding for the types and fixing the errors in Feature B, all in an effort to make sure Feature B can be compiled even if your tests do not depend on Feature B at all. What an frustration indeed!

Please note that I do not endorse scattering wrong code everywhere irresponsibly.  However, in early prototyping, it is very difficult to nail down the system design and API before doing a bit of trial-and-errors. In sandbox playground stage,

### Test Test Test!
Because of the dynamic nature python, I  really felt compelled to embrace test driven development (TDD). So I started to write the unit tests and functional tests at a much earlier stage of development than I usually do. When writing the test suites, I became the first consumers of the API of the code base. I immediately got a sense of whether the API worked or failed totally. This provided an opportunity to spot design issues much earlier, long before I invested a lot of effort implementing the API. Once the test suites was stabilized, I just followed them as the guidelines and the actual coding process just felt like a breeze.

Initially I was very concerned that without the type checking in static typing languges, I might have planted a lot bugs inadvently. As it turns out, this is not a big problem. I am pretty familiar with the business logics so I was able to provide a very comprehensive test suites. The test suites successfully caught several bugs in development and no more issues were found in the user testing later.

## My Gripes



### Option Type
There is no form option type in python. As an alternative, you can define a function that return a meaningful value or return `None`. As a consumer of such function, I can branch the logics depending on whether it is `None`. This approach meets my needs. However, my IDE [PyCharm](http://www.jetbrains.com/pycharm/) shows no affection. PyCharm will provide no auto completion for the methods of the returned value because it can be `None`.

Not sure who I should blame, python or PyCharm?

### Refactoring
If you only need to rename a variable or change the signature of a method, an IDE like PyCharm should be able to help you without much drama. However, if you need to change some structural change, you are on your own. Suppose you remove a method from a class and replace with something else. With a static typed language, the IDE will sound the fire alarm everywhere such method is used and you can quickly go there and correct the errors. With Python, the IDE is not able to give you any alert because of dynamic typing. The issue will only pop up when you run the test, which is very distracting.

Type hinting will be introduced in Python 3.5. Let's see if it could alleviate this problem to some extent.

### Minor Nuisances
There are few minor nuisances that are not harmful for my productivity but really bug me.

* Class naming follows camelCase; but variable and method naming are done with lowercase and underscore as separator.
* No method overloading.

## Wrap Up
