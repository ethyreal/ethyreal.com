
## Dependiency Injection 

from 2017-0703

Explicit v Implcicite
Constructor v Property
...just one more argument

i wonder if it would be better to start using mutating properties on the struct and reduced the number of constructor arguments?

I love constructor arguments as they are explicit dependencies, which are a good.  I think we should question if every property an object has needs to be in the constructor.  The only real requirement to configure a section is an array of items right?  maybe  all we need on init is just the items:

```
init(items:[Item]) { }
```

Instead of a giant constructor:

```
    func makeSectionDescriptor(with group: TaskGroup) -> ListSectionDescriptor<ContactTask> {
        let tasks = group.tasks
        let title = group.title
        let header = self.makeSectionHeader(with: title, group: group)
        return ListSectionDescriptor<ContactTask>(items: tasks, headerView: header, footerView: nil, autoSizingHeader: false)
    }
```

We only init with required items then assign any extra config we need:

```
    func makeSectionDescriptor(with group: TaskGroup) -> ListSectionDescriptor<ContactTask> {

        let section = ListSectionDescriptor<ContactTask>(items: group.tasks)
        section.headerView = self.makeSectionHeader(with: group.title, group: group)
        section.autoSizingHeader = false
        return section
    }
```


## ObjC Delegate Pattern / Convention

from 2017-07-12

```objective-c
- (void)personalInfoViewControllerDidTapEditContact;
```

can this delegate follow the pattern above?

this is described in our objective-c style guide [protocols section](https://stash.sv2.trulia.com/projects/ZRM/repos/mob-truliaagent/browse/docs/style-guide-objc.md#protocols) :

## Protocols

In a [delegate or data source protocol](https://developer.apple.com/library/ios/documentation/General/Conceptual/CocoaEncyclopedia/DelegatesandDataSources/DelegatesandDataSources.html), the first parameter to each method should be the object sending the message.

This helps disambiguate in cases when an object is the delegate for multiple similarly-typed objects, and it helps clarify intent to readers of a class implementing these delegate methods.

**For example:**

```objective-c
- (void)tableView:(UITableView *)tableView didSelectRowAtIndexPath:(NSIndexPath *)indexPath;
```

**Not:**

```objective-c
- (void)didSelectTableRowAtIndexPath:(NSIndexPath *)indexPath;
```


## Clear Method Naming

from  2017-07-14

```swift
extension String {
    func trimSuffix(suffix:String) -> String {
        return self.replacingOccurrences(of: suffix, with: "")
    }
}
```

technically this will trim whatever is passed in, be it suffix or prefix and in the middle. maybe the name should reflect that? or it could be changed to only trim the suffix?


## Terenary + Nil Coelessing


```swift
self.contact = contact == nil ? inboxItem?.firstContact() : contact
```

Minor, but this could be simplified

```swift
 self.contact = contact ?? message?.firstContact()
```

- or:

```swift
guard let status = someFeature?.status else { return .none }
return status
```

could just be

```swift
return someFeature?.status ?? .none   
```