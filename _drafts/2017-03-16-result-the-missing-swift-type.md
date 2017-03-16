---
title:  "Result the Missing Swift Type"
date:   2017-03-16 01:30:00 -0800
categories: swift functional result
---


Simple enum, much like Optional:

```swift

enum Result<T> {
    
    case failure(Error)
    case success(T)
}

```

Add some helpers to get common values:

```swift
extension Result {
    
    var isSuccess:Bool {
        switch self {
        case .success: return true
        case .failure: return false
        }
    }
    
    var isFailure:Bool {
        switch self {
        case .success: return false
        case .failure: return true
        }
    }
    
    var error:Error? {
        switch self {
        case let .failure(error):
            return error
        case .success:
            return nil
        }
    }
    
    var value:T? {
        switch self {
        case let .success(payload):
            return payload
        case .failure:
            return nil
        }
    }
}
```

Add monadic functions to enable chaining results together:

```swift

extension Result {

    func flatMap<U>(_ transform:(T) -> Result<U>) -> Result<U> {
        
        switch self {
        case let .failure(error):
            return .failure(error)
        case let .success(value):
            return transform(value)
        }
    }
    
}

```


For those times we need to drop into caveman debuging mode and print these out, we can conform to ```CustomDebugStringConvertible````:.


```swift

extension Result: CustomDebugStringConvertible {
    
    var debugDescription: String {
        switch self {
        case let .failure(error):
            return "Result.failure( error: \(error))"
        case let .success(value):
            return "Result.success( \(value) )"
        }
    }
}
```
