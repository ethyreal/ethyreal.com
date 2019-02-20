---
title:  "The Mighty Container"
date:   2019-02-20 08:01:00 -0800
categories: composition functional swift
---

> This is part of a remix of the [Mostly Adequate Guide](https://github.com/MostlyAdequate/mostly-adequate-guide) (Chapter 8).  I have translated examples into swift and updated text where appropriate.

We've seen how to write programs which pipe data through a series of pure functions. They are declarative specifications of behaviour. But what about control flow, error handling, asynchronous actions, state and, dare I say, effects?! In this chapter, we will discover the foundation upon which all of these helpful abstractions are built.

First we will create a container. This container must hold any type of value; a ziplock that holds only tapioca pudding is rarely useful. It will be an object, but we will not give it properties and methods in the OO sense. No, we will treat it like a treasure chest - a special box that cradles our valuable data.

```swift
struct Container<A> {

    let value:A

    static func of(_ value:A) -> Container<A> {
        return Container(value: value)
    }
}
```

Here is our first container. We've thoughtfully named it `Container`. We will use `Container.of` as a constructor which adds a little sematics. There's more to the `of` function than meets the eye, but for now, think of it as the proper way to place values into our container.

Let's examine our brand new box...

```swift
Container.of(42)
// Container<Int>(value: 42)

Container.of("hotdogs")
// Container<String>(value: "hotdogs")

Container.of(Container.of(["name": "yoda"]))
// Container<Container<Dictionary<String, String>>>(value: Container<Dictionary<String, String>>(value: ["name": "yoda"]))
```

Let's make a few things clear before we move on:

* `Container` is an object with one property. Lots of containers just hold one thing, though they aren't limited to one. We've arbitrarily named its property `value`.
* The `value` cannot be one specific type or our `Container` would hardly live up to the name.
* Once data goes into the `Container` it stays there. We *could* get it out by using `.value`, but that would defeat the purpose.

The reasons we're doing this will become clear as a mason jar, but for now, bear with me.

## My First Functor"

Once our value, whatever it may be, is in the container, we'll need a way to run functions on it.

```swift
extension Container {

    // (a -> b) -> Container a -> Container b
    func map<B>(_ t:(A) -> B) -> Container<B> {
        return Container<B>.of(t(value))
    }
}
```

Why, it's just like Array's famous `map`, except we have `Container a` instead of `[a]`. And it works essentially the same way:

```swift
Container.of(2).map { $0 + 2 }
// Container<Int>(value: 4)

Container.of("flamethrowers").map { $0.uppercased() }
// Container<String>(value: "FLAMETHROWERS")

Container.of("bombs")
    .map { "\($0) away" }
    .map { $0.count }
// Container<Int>(value: 10)
```

We can work with our value without ever having to leave the `Container`. This is a remarkable thing. Our value in the `Container` is handed to the `map` function so we can fuss with it and afterward, returned to its `Container` for safe keeping. As a result of never leaving the `Container`, we can continue to `map` away, running functions as we please. We can even change the type as we go along as demonstrated in the latter of the three examples.

Wait a minute, if we keep calling `map`, it appears to be some sort of composition! What mathematical magic is at work here? Well chaps, we've just discovered *Functors*.

> A Functor is a type that implements `map` and obeys some laws"

Yes, *Functor* is simply an interface with a contract. We could have just as easily named it *Mappable*, but now, where's the *fun* in that? Functors come from category theory and we'll look at the maths in detail toward the end of the chapter, but for now, let's work on intuition and practical uses for this bizarrely named interface.

What reason could we possibly have for bottling up a value and using `map` to get at it? The answer reveals itself if we choose a better question: What do we gain from asking our container to apply functions for us? Well, abstraction of function application. When we `map` a function, we ask the container type to run it for us. This is a very powerful concept, indeed.

> Working code can be found in the Chapter 8 tests of the [Mostly Adequate Swift framework](https://github.com/ethyreal/mostly-adequate-guide-swift)
