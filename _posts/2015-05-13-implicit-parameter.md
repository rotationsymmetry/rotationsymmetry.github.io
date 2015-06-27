---
layout: post
title: Implicit Parameter
comments: true
categories: Scala
---

Scala allows function to take implicit parameters. The implicit parameters are usually objects that are needed by a series of functions to perform their duties. When an object of appropriate type is available in the scope, all the functions can receive this object implicitly, making the code more concise.

<!-- more -->

Let's look at an example. Suppose you write code for the logistics department of a company. The employee of the logistics department are shippers who will send and receive items. The shipping service are provided by Fedex, UPS or USPS.

``` scala
case class Item(name: String)
case class ShippingService(name: String)

case class Shipper(name :String) {
	def send(item: Item, to: ShippingService)
	def receive(from: ShippingService): Item
}
```

Suppose in this season, the Fedex is the most cost effective vendor. Then Tom, the shipper, will send and receive all items through Fedex.

``` scala
val tom = Shipper("Tom")
val fedex = ShippingService("Fedex")

tom.send(Item("Apple"), fedex)
tom.send(Item("Orange"), fedex)
tom.send(Item("Banana"), fedex)
val receivedItem = tom.receive(fedex)
```

Obviously, it will become quite tedious to type fedex in every line of the code. So this is the ideal use case for implicit parameters. We will revise the `Shipper` class as follows:

``` scala
case class Shipper(name) {
	def send(item: Item)(implicit to: ShippingService)
	def receive(implicit from: ShippingService): Item
}
```

Before Tom before going through the shipping list, we declare Fedex as an implicit in the scope of the code. Then Tom's daily chores become a lot simpler:

``` scala
val tom = Shipper("Tom")
implicit val fedex = ShippingService("Fedex")

tom.send(Item("Apple")) // same as tom.send(Item("Apple"))(fedex)
tom.send(Item("Orange"))
tom.send(Item("Banana"))
val receivedItem = tom.receive
```

Please note that you are bound to use the implicit object in your scope every time you call `send` or `receive`. Instead you can instantly override the implicit by providing a `ShippingService` argument. No changes to the `Shipper` is needed.

``` scala
val tom = Shipper("Tom")
implicit val fedex = ShippingService("Fedex")

tom.send(Item("Apple")) // use Fedex
tom.send(Item("Piano"))(ShippingService("USPS")) // USPS is cheaper for large items.
```

So implicit is really a nice feature of Scala that will make your code more concise without sacrificing flexibility.

Before you accuse me of drinking too much implicit Kool-Aid, I strongly encourage you to think twice before declaring implicit parameters. For example, if you hand the one liner `tom.send(Item("Apple"))` to a colleague, there is absolutely no way for him to know `send` takes an implicit parameter. Without a trip to source code of `Shipper`, he is likely to scratch his head wondering where the `Item("Apple")` is sent to.

My recommendation for introducing implicit to your project: use implicit sparsely, and when you do, document where implicits are used at the beginning of the your README.

PS: I have written another [blog post](/2015/05/10/when-to-use-curried-function/) to discuss why you will use curried function for implicit parameters.
