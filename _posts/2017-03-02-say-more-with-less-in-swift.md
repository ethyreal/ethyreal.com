---
layout: single
title:  "Say More With Less In Swift"
date:   2017-03-01 09:43:00 -0800
categories: swift style
---
For many years of iOS development each team I have worked with has adobted a style guide usually based off of Apple's [Cocoa Coding Guidelines](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/CodingGuidelines/CodingGuidelines.html).  Every team has their own style but mostly they all have been inspired by Cocoa and hence are very descriptive.  Over time I grew to love this, it was clear and self documenting if at the cost of being especially verbose.  More than anything I always wanted my code to not look and feel any different from Apple's code.   This changed last summer with the introduciton of swift 3 and Swift's [API Design Guidelines](https://swift.org/documentation/api-design-guidelines/).

Key takeaways for me are:

- Favor clearity at the call site
- Say more with less

The second point for me is the trickiest.  Comming from Cocoa API's brevity and clarity seem in opposition.  However by removing needless words, adding call site context and learning Swift's new vocabulary we can actually "Say more with less".

As an example let say as a user I want to see a list of great musicians that died last year. 

First we might start with a simple model:

```swift
struct Person {
    let name:String
    let age:Int
}
```

We might need to load this model from a restful service that provides a JSON array.  After decoding it might be represented like this:

```swift
let remoteData:[[String: Any]] = [
    ["name" : "David Bowie", "age" : 69],
    ["name" : "Prince", "age" : 57],
    ["name" : "Leonard Cohen", "age" : 82],
    ["name" : "George Michael", "age" : 52]
]
```

This is straighforward enough, an array of dictionaries that have string keys and any value.  Mentally though reading the desclaration we have to compute "remoteData" equals and array of "string:any" dictionaries.  If all our JSON will look like this could this be made more clear with a ```typealias``` for our JSON:


```swift
typealias JSONDictionary = [String: Any]
```

We could additionaly rename the variable to something that is more specific in descibeing its contents:
 
```swift
let jsonArray:[JSONDictionary] = [
    ["name" : "David Bowie", "age" : 69],
    ["name" : "Prince", "age" : 57],
    ["name" : "Leonard Cohen", "age" : 82],
    ["name" : "George Michael", "age" : 52]
]
```

We could extend our ```Person``` model to understand how to construct ifself from a ```JSONDictionary```:

```swift
extension Person {
    
    init?(json:JSONDictionary) {
        guard let name = json["name"] as? String else { return nil }
        guard let age = json["age"] as? Int else { return nil }
        
        self.init(name:name, age:age)
    }
}
```

We do this inside an extension so we don't loose our default struct initializer.

Now we use our trusty for loop to build and array of persons:

```swift
var people = [Person]()

for json in jsonArray {
    if let person = Person(json: json) {
        people.append(person)
    }
}
```

This is familiar and clear.  It's clear we are looping through and array of JSON, and building an array of ```Person```'s.  We've all done this many many times.

This is infact done so much that there are actually methods to do exacly that.  Transforming one collection into another.  Enter our friends ```map()``` and ```flatMap```.  Each of these takes a transform closure to apply to each element of the collection being mapped over.  So we can write the same logic:

```swift
let people:[Person] = jsonArray.flatMap({ (json) -> Person? in
    return Person(json: json)
})
```

This still seems clear and DRY which is good.  There is a lot of syntax in this that while clear could likely be assumed based on context.  We could rewrite this using swift's trailing closure syntax, and omit the return type:

```swift
let people:[Person] = jsonArray.flatMap { (json) in
    return Person(json: json)
}
```

Still clear right?  We haven't lost any meaning and we've reduced the ammount of symbols we need to process in our heads.

We see at the call site that we are mapping an array of JSON, then passing that JSON to an arguement called "json" so adding the ```json``` label in the closure seems like its duplicating words.  In closures we do have the agrument list available with ```$0, $1, etc```.  On their own they seem opaque and terse but look at the call site now:

```swift
let people:[Person] = jsonArray.flatMap {
    return Person(json: $0)
}
```

If there is only one line in an expresion that returns a value then it can be assumed to return, could we remove the return?  While we are at it, does this read well on one line?

```swift
let people:[Person] = jsonArray.flatMap { Person(json: $0) }
```

We could even do a step further and drop the type from the instance variable:

```swift
let people = jsonArray.flatMap { Person(json: $0) }
```

With good meaningful varivables and call site context this is still clear at first glance what is happening.  The irony of using a lot of words to promote less words in code.



