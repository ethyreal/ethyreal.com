---
layout: single
title:  "Working with UIFont"
date:   2012-11-01 09:43:00 -0800
categories: ios fonts
---
I have been working with fonts in iOS quite a bit recently and wanted to gather what I have learned.

For UIKit elements you need to use UIFont.

Creating a UIFont is very simple, there are serveral easy to use class methods to conveniently instantiate and instance:

```swift
[UIFont systemFontOfSize:12.0]; // Helvetica Regular
 
[UIFont boldSystemFontOfSize:12.0]; //Helvetica Bold
    
[UIFont italicSystemFontOfSize:12.0]; //Helvetica Italic
```

If you want a different font face you can use:

```swift
[UIFont fontWithName:@"ArialMT" size:12.0]; // Arial Regular
```

The problem with this is fontWIthName:size: doesn't take a family name like just Arial, it needs the "real" font name or the postscript font name. Luckily UIFont two other nice class methods to get an array of family names and an array of font names for a given family:

```swift
NSArray *familyNames = [UIFont familyNames];
    
NSArray *arialFontNames = [UIFont fontNamesForFamilyName:@"Arial"];
```

So given these methods we can log out a list of all the "real" font names with something like this:

```swift
// List all fonts on device
NSArray *familyNames = [[NSArray alloc] initWithArray:[UIFont familyNames]];
NSArray *fontNames;
NSInteger indexFamily, indexFont;
for (indexFamily=0; indexFamily<[familyNames count]; ++indexFamily)
{
  NSLog(@"Family name: %@", [familyNames objectAtIndex:indexFamily]);
  fontNames = [[NSArray alloc] initWithArray:[UIFont fontNamesForFamilyName:[familyNames objectAtIndex:indexFamily]]];
        
  for (indexFont=0; indexFont<[fontNames count]; ++indexFont)
  {
    NSLog(@"    Font name: %@", [fontNames objectAtIndex:indexFont]);
  }
  [fontNames release];
}
[familyNames release];
```

Using the output above we can find the font name we need and create a UIFont for any UILabel or any other UIKit element that takes a UIFont.

