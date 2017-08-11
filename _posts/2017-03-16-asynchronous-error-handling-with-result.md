---
title:  "Asynchronous Error Handling with Result"
date:   2017-03-16 01:30:00 -0800
categories: swift error functional result
---

Handling error's in swift is fairly straighforward.  Let say we have an artist:

```swift
struct Artist {
    let name:String
    let age:Int
}
```

That artist needs to be persisted, so we add a service:

```swift
class ArtistService {
    
    func save(_ artist:Artist) throws  {
        // persist somehow...       
    }
}
```

Because things can go wrong and need to be able handle those cases so we add an error:

```swift
enum ArtistError: Error {
    case parsing
    case network
    case persistance
}
```

Now we can create and save an artist all the while feeling comfortable in that we have handled our edge cases:

```swift
let service = ArtistService()

let bowie = Artist(name: "David Bowie", age: 69)

do {
    try service.save(bowie)
} catch ArtistError.parsingError {
    // handle failed to parse
}
```

What happens though when we need to load artists?  Most likely that will take time, either querying a local database or fetching from a network.  Eitherway we would normally wind of with an API using a callback that provides the requested resource and an error:

```
class ArtistService {
    
    func artists(completion: ([Artist]?, Error?) -> Void) {
      // fetch inspirational people   
        
    }
}
```

Safely handling this always feels awkward though because even since we are using two optional values to represent this we actually have four possible states:

```
service.artists { (artists, error) in

    if error != nil, artists == nil {
        // error and no artits cool, handle error
    }
    else if error != nil, artists != nil {
        // error but we got artists...strange.. but lets handle error
    }
    else if artists != nil, error == nil {
        // we got artists and no error yay!!
    }
    else if artists == nil, error == nil {
        // no artists and no error...um..so now what?
    }
}
```

This seems too complicated.  What we really want to represent is just two states here: success and failure.  Luckily swift provides enums to the reseue!  Enums are basically swifts implemetation of a Sum Type of [Tagged Union](https://en.wikipedia.org/wiki/Tagged_union).  We start simple and build one just for our artists request:

```
enum ArtistsResult {
    
    case failure(Error)
    case success([Artist])
}
```

Update our service to use our fancy result:

```
class ArtistService {
    
    func artists(completion: (ArtistsResult) -> Void) {
           // fetch inspirational people   
    }
}
```

Now using it we can cleanly just deal with two possible outcomes:

```
service.artists { result in
    switch result {
    case .failure( let error ):
        // oh no, total fail!! handle the error
        
    case .success( let artists ):
        // nice!
    }
}
```

I feel good about this! no edge cases, cleanly handle two possible options.  It doesn't seem very resuable though, it only handles an array of artists.  We could add another result for a single artist right?

```
enum ArtistResult {
    
    case failure(Error)
    case success(Artist)
}
```

But then we are defining an enum each time we want to handle another asynchronous operation that seems just as bad as handling all the cases of two seperatate types in the if/else soup.  If the type could be generic over all possible successes than we'd be in business:

```
enum Result<T> {
    
    case failure(Error)
    case success(T)
}
```

Our signature and call site look similar:

```
class ArtistService {
    
    func artists(completion: (Result<[Artist]>) -> Void) {
           // fetch inspirational people   
    }
}

service.artists { result in
    switch result {
    case .failure( let error ):
        // oh no, total fail!! handle the error
        
    case .success( let artists ):
        // nice!
    }
}
```

Now we have a ```Result<T>``` type we can reuse for all our asynchronous error handling!  It explicitly defines two possile options and each option can be clearly handled using a simple case statement.

Many others have [written](https://www.cocoawithlove.com/blog/2016/08/21/result-types-part-one.html) and [spoken](http://2014.funswiftconf.com/speakers/john.html) in great detail about using ```Result``` for fun an profit


