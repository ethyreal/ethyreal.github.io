---
title:  "Functional Husbandry"
date:   2019-02-09 18:28:00 -0800
categories: composition functional swift
---

> This is part of a remix of the [Mostly Adequate Guide](https://github.com/MostlyAdequate/mostly-adequate-guide) (Chapter 5).  I have translated examples into swift and updated text where appropriate.

Here's _compose_ :

```swift
precedencegroup BackwardsComposition {
    associativity: left
}

infix operator <<<: BackwardsComposition

func <<< <A, B, C>(_ f: @escaping (B) -> C, _ g: @escaping (A) -> B) -> (A) -> C {
    return { x in f(g(x)) }
}
```

... Don't be scared! This is the level-9000-super-Saiyan-form of _compose_. For the sake of reasoning, let's drop the infix implementation and consider a simpler form that can compose two functions together. Once you get your head around that, you can push the abstraction further and consider it simply works for any number of functions (we could even prove that)!

Here's a more friendly _compose_ for you my dear readers:

```swift
func compose<A, B, C>(_ f: @escaping (B) -> C, _ g: @escaping (A) -> B) -> (A) -> C {
    return { x in
        f(g(x))
    }
}
```

`f` and `g` are functions and `x` is the value being "piped" through them.  The generics `A`, `B` and `C` allow both `f` and `g` to transform any type into any other preserving type safety.

 Composition feels like function husbandry. You, breeder of functions, select two with traits you'd like to combine and mash them together to spawn a brand new one. Usage is as follows:

```swift
let toUpperCase = { (x:String) in x.uppercased() }
let exclaim = { (x:String) in "\(x)!" }
let shout = compose(exclaim, toUpperCase)

shout("send in the clowns") // "SEND IN THE CLOWNS!"
```

 The composition of two functions returns a new function. This makes perfect sense: composing two units of some type (in this case function) should yield a new unit of that very type. You don't plug two legos together and get a lincoln log. There is a theory here, some underlying law that we will discover in due time.

 In our definition of `compose`, the `g` will run before the `f`, creating a right to left flow of data. This is much more readable than nesting a bunch of function calls. Without compose, the above would read:

```swift
let shout = { (x:String) in exclaim(toUpperCase(x)) }
```

 Instead of inside to outside, we run right to left, which I suppose is a step in the left direction (boo!). Let's look at an example where sequence matters:

```swift
func head(_ x:[String]) -> String {
    return x.first ?? ""
}

func reverse(_ x:[String]) -> [String] {
    return x.reversed()
}

let last = compose(head, reverse)

last(["jumpkick", "roundhouse", "uppercut"]) // "uppercut"
```

 `reverse` will turn the list around while `head` grabs the initial item. This results in an effective, albeit inefficient, `last` function. The sequence of functions in the composition should be apparent here. We could define a left to right version, however, we mirror the mathematical version much more closely as it stands. That's right, composition is straight
from the math books. In fact, perhaps it's time to look at a property that holds for any composition.

```swift
 // associativity
 compose(f, compose(g, h)) === compose(compose(f, g), h)
```

 Composition is associative, meaning it doesn't matter how you group two of them. So, should we choose to uppercase the string, we can write:

```swift
compose(toUpperCase, compose(head, reverse))
// or
compose(compose(toUpperCase, head), reverse)
```

 Since it doesn't matter how we group our calls to compose, the result will be the same. That allows us to use our infix verson of our compose method:

```swift
// previously we'd have to write two composes, but since it's associative,
// we can give compose as many fn's as we like and let it decide how to group them.

let arg = ["jumpkick", "roundhouse", "uppercut"]
let lastUpper = toUpperCase <<< head <<< reverse
let loudLastUpper = exclaim <<< toUpperCase <<< head <<< reverse

lastUpper(arg) // 'UPPERCUT'
loudLastUpper(arg) // 'UPPERCUT!
```

 Applying the associative property gives us this flexibility and peace of mind that the result will be equivalent.

 One pleasant benefit of associativity is that any group of functions can be extracted and bundled together in their very own composition. Let's play with refactoring our previous example:

```swift
let loudLastUpper = exclaim <<< toUpperCase <<< head <<< reverse

// -- or ---------------------------------------------------------------

let last = head <<< reverse
let loudLastUpper = exclaim <<< toUpperCase <<< last

// -- or ---------------------------------------------------------------

let last = head <<< reverse
let angry = exclaim <<< toUpperCase
let loudLastUpper = angry <<< last
```

 There's no right or wrong answers - we're just plugging our legos together in whatever way we please. Usually it's best to group things in a reusable way like `last` and `angry`. If familiar with Fowler's "[Refactoring](https://martinfowler.com/books/refactoring.html)", one might recognize this process as "[extract function](https://refactoring.com/catalog/extractFunction.html)" ...except without all the object state to worry about.

> Working code can be found in the Chapter 5 page of the [Mostly Adequate Swift Playground](https://github.com/ethyreal/mostly-adequate-guide-swift)
