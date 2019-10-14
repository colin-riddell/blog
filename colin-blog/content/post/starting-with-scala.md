---
title: "Starting With Scala"
date: 2019-09-24T21:52:10+01:00
draft: false
author: Colin Riddell
description : ""
slug : ""
tags : ["scala", "functional", "JVM", "asynchronous"]
categories : []
externalLink : ""
series : [scala]
---

Scala is a functional programming language.

For years, I've always shied away from functional programming languages. I've done that because I always thought they were only really used by hard-core academic *computer scientists* to make themselves sound intelligent in front of others.

I've found out recently there are some really fun and powerful things to be learned from functional languages, such as Scala. They're fast, good for asynchronous operations, have low latency and because of that they are becoming more prevalent and sought after to meet the real-time, low-latency demands of today.

ReactiveX, Observables and event-driven architectures are all important technologies and concepts we can't ignore - and they borrow from functional programming.


I'm going to learn more about functional languages and share what strikes me as interesting about them, as I try to pick them up.

```
  _________             .__          
 /   _____/ ____ _____  |  | _____   
 \_____  \_/ ___\\__  \ |  | \__  \  
 /        \  \___ / __ \|  |__/ __ \_
/_______  /\___  >____  /____(____  /
        \/     \/     \/          \/
```

## Why Scala
Not sure when it first surfaced or became popular - but before getting interested I think it's worth asking **why?**


🧐 ... Well, I'm still not entirely sure why - but am going to take a stab at it.

### Scala is Fast
Scala runs on the Java Virtual Machine - before you get your pitchfork out, just, let me explain. Applications running on the JVM are not slow. The JVM merely has a slow startup time. Execution of code once the JVM has fired up is relatively fast.


![](https://jazzy.id.au/images/blogs/javascalabenchmark/scalavsjava.png)
(DZone, 2012, [https://dzone.com/articles/benchmarking-scala-against](https://dzone.com/articles/benchmarking-scala-against))


### Scala is good for Asynchronous tasks

Well, Scala has promises. We're familiar with promises from **JavaScript**. To recap, a promise is a placeholder object that's given to the caller of a function or operation that's not yet returned with the completed operation.

Think of a promise as being like a note with your order number on it that you get when you order a coffee in a café. You're very seldom handed your order immediately, so while it's being made they give you a note. A note that *promises* that you'll be given your coffee when it's ready.


With Scala, since it's working on the back-end the asynchronous stuff happens when doing some database transactions, or making further requests to other services.


## Scala is functional and Object Orientated language

Scala allows you to do functional programming. This is a function that takes a number and adds one to it. It's assigned to the variable `addOne`

```scala
var addOne = (x: Int) => x + 1
```
Some things I noticed when playing around with simple functions like this in **scala**

* **A function definition must be assigned to something to be useful.** It's possible to have the `(x: Int) => x + 1` as an unnamed function, but that on its own isn't useful and the compiler gives a warning
`a pure expression does nothing in statement position`.
* **You can do multi-line functions brackets {}.** Wrapping the code to the right of the `=>` in a block gives us the ability to do multi-line functions.. But I get the impression that it's **not cool** to use them!
<iframe height="200px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=JXnfQGX/1&layout=h58"></iframe>

<!--```scala
var addOne = (x: Int) => {
  printf("this is my multi-line scala function")
  x + 1
}

```-->
* **Brackets {} in multi-line functions are not needed** But I think it looks nasty without them.
* **The last expression in a function is implicitly returned** Being new to functional languages, I don't yet know if this a typically functional thing, or if it's a Scala only thing. But in our above code, as `x+1` is the first and last expression in that function, it's returned.  If we had something like the following, the result is 75, as the result of the last expression is what's returned from the function.
<iframe height="200px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=biUc0x4/1"></iframe>
* **Functions and methods are different**
   * Functions can't have a `return` statement. Methods can.
   * Functions can't have multiple parameter lists. Methods can.
   * Methods can have no parameters at all, if required.
   * Methods syntax requires the `def` keyword and look similar to python or ruby.
<iframe height="200px" frameborder="0" style="width: 100%" src="https://embed.scalafiddle.io/embed?sfid=eso2fGg/1"></iframe>

### Scala has classes

Classes in scala take their constructor arguments along with the class name
```scala
class Greeter(prefix: String, suffix: String) {
  def greet(name: String): Unit =
    println(prefix + name + suffix)
}
```

Objects of a class are instantiated with `new`

```scala
val greeter = new Greeter("Mr", "Riddell")
greeter.greet("Colin")
```

## Scala Seems fun

Going to investigate using Scala for web applications and try to learn more about it. In the next part I'll share more findings.
