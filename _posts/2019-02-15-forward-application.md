---
title:  "Forward Application"
date:   2019-02-15 09:17:00 -0800
categories: composition functional swift
---

 Previously we built up [compositions of functions](/composition/functional/swift/forward-composition/).  These created piplines to pass data through.  The data though is always an after thought though.  Semantically we describe an action or series of actions then we say what those actions are going to apply to.

 Using the previous example we had a list of moves
 
 ```swift
 let moves = ["jumpkick", "roundhouse", "uppercut"]
```

We want to loudly declare a move, in this case the last one:

```swift
 let loudLastUpper = reverse 
                        >>> head 
                        >>> toUpperCase 
                        >>> exclaim
loudLastUpper(moves) //"UPPERCUT!"
```

This is a familiar paradigm: `action` => `data`.  Action or function is applied to some data.  Programming 101 right?  But what if we thought about our data first?  Could we apply our moves to the `loudLastUpper` action?  We could define an `apply` function:

```swift
func apply<A, B>(_ x: A, to f: (A) -> B) -> B {
    return f(x)
}
```

Now let's _apply_ it to the previous example:

```swift
apply(moves, to: loudLastUpper)  //"UPPERCUT!"
```

This is different, but still understandable right?  Apply these moves to the `loudLastUpper` action.

We could inline the moves list and it still reads well:

```swift
apply(["jumpkick", "roundhouse", "uppercut"],
      to: loudLastUpper)
```

This is almost what we are after.  What if we infixed this apply function so we had somthing like `moves applyTo loudLastUpper`.  This seems similar to what unix does with the pipe (`|`) operator: 

```swift
// find all the `swift` files on a computer:
locate "*.swift" | grep swift
```

Unix pipes or `applies` the results of `locate` to `grep` two totally seperate commands are chained or composed together!

How about a similar oppertor for swift, we can borrow from other and use the `|>` operator used in F# and Haskel:

```swift
precedencegroup ForwardApplication {
    associativity: left
}

infix operator |>: ForwardApplication

public func |> <A, B>(a: A, f: (A) -> B) -> B {
    return f(a)
}
```

Same goodness of `apply(to:)` but now are an use it `infix`!

```swift
moves |> loudLastUpper //"UPPERCUT!"
```

While this is a new operator to get used to, it is easy to undersand even thought knowing F# or Haskel, we are just piping data through!

We might not even need our `loudLastUpper` function any more, if it's only used here it seems a waste.  Could we just inline that:

```swift
moves |> reverse
        >>> head
        >>> toUpperCase
        >>> exclaim
```

Out of the box this isn't going to work.  We have conflicting precedence groups.  We need to define an order of operations (just like math!).  Here we have both `ForwardComposition` and `ForwardApplication` with `left` associativity.  We could use our friend parenthesis to group the composition first:

```swift
moves |> (reverse
            >>> head
            >>> toUpperCase
            >>> exclaim)
```

This works, but adds noise and cognative overhead to our smoothly read composition.  In some situations we may want that, but in others we might want a rule.  If we are doing forward application and composition in the same place it might make sense to compose first as a rule.  We can change our`ForwardComposition` precedence group to be placed higher than our `ForwardApplication`:

```swift
precedencegroup ForwardComposition {
    associativity: left
    higherThan: ForwardApplication
}
```

Now this works with no complaints:

```swift
moves |> reverse
        >>> head
        >>> toUpperCase
        >>> exclaim
```

Now you might be wondinging how different this `apply` (`|>`) function is from `compose` (`>>>`)?  On the surface they are very similar, apply puts a function's argument in front where as compose calles one function with the result of another:

```swift
public func |> <A, B>(a: A, f: (A) -> B) -> B {
    return f(a)
}
```
Vs:

```swift
func >>> <A, B, C>(_ f: @escaping (A) -> B, _ g: @escaping (B) -> C) -> (A) -> C {
    return { x in g(f(x)) }
}
```

In some cases (like our example) you could easily replace one with the other:

```swift
["jumpkick", "roundhouse", "uppercut"]
    |> reverse
    |> head
    |> toUpperCase
    |> exclaim
// "UPPERCUT!"

// same result as:

["jumpkick", "roundhouse", "uppercut"]
    |> reverse
    >>> head
    >>> toUpperCase
    >>> exclaim    
// "UPPERCUT!"
```

In this use case I personally find the piped version easier to read than the composed. For inline logic or one offs this is great, easy to visualize. This doesn't mean compose doesn't have its uses too. Being able to form new functions, save them, pass them around is an incredibly powerful thing.

> Working code can be found in the Chapter 5 page of the [Mostly Adequate Swift Playground](https://github.com/ethyreal/mostly-adequate-guide-swift)


