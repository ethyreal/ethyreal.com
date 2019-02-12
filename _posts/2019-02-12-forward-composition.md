---
title:  "Forward Composition"
date:   2019-02-12 09:32:00 -0800
categories: composition functional
---

In the previous post I "remixed" a portion of the [composition chapter](/composition/functional/functional-husbandry/) in the [Mostly Adequate Guide](https://github.com/MostlyAdequate/mostly-adequate-guide).  While reading that chapter and later translating the direction of composition kept confusing me.  I wanted to convey the idea's as close as possible.. keeping all the great humor like:
>Instead of inside to outside, we run right to left, which I suppose is a step in the left direction (boo!). 

And this does make sense, it reads right to left:

```swift
let  = exclaim <<< toUpperCase <<< head <<< reverse
loudLastUpper(["jumpkick", "roundhouse", "uppercut"]) //"UPPERCUT!"
```

But it doesn't feel like a huge readability gain from:

```swift
let loudLastUpper = { (x:[String]) in  exclaim( toUpperCase( head( reverse(x) ) ) ) }
```

Or the more common:

```swift
func loudLastUpper(_ x:[String]) -> String {
    return exclaim( toUpperCase( head( reverse(x) ) ) )
}
```

It's less parentheses and I do like the direction arrows pointing out the flow of data.  We are still reading right to left which for me doesn't feel natual in a language that is written left to right.

What I've been usually using is a `ForwardComposition` like this:

```swift
precedencegroup ForwardComposition {
    associativity: left
}

infix operator >>>: ForwardComposition

func >>> <A, B, C>(_ f: @escaping (A) -> B, _ g: @escaping (B) -> C) -> (A) -> C {
    return { x in g(f(x)) }
}
```

I've heard this called `composeRight` or it could be a regular function called `forwardCompose` instead of an infix:

```swift
func forwardCompose<A, B, C>(_ f: @escaping (A) -> B, _ g: @escaping (B) -> C) -> (A) -> C {
    return { a in
        g(f(a))
    }
}
```

Either way we can chain or pipe the logic from the left to the right, the same direction as the language we are writing in!

```swift
// same result as before!

let loudLastUpper = reverse >>> head >>> toUpperCase >>> exclaim
loudLastUpper(["jumpkick", "roundhouse", "uppercut"]) //"UPPERCUT!"
```

Lastly we can break them up with line breaks to increase readability further... especially in cases of longer method names:

```swift
let loudLastUpper = reverse
                    >>> head
                    >>> toUpperCase
                    >>> exclaim
```


