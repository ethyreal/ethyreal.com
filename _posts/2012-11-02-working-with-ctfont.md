---
title:  "Working with CTFont"
date:   2012-11-02 09:43:00 -0800
categories: ios fonts
---
In a previous post I went through [creating UIFonts](/ios/fonts/working-with-uifont/). UIFonts are great for UIKit classes but when you want rich text and need to set fonts for NSAttributedStrings and/or layout your text with core text you need to use CTFonts.

Creating a CTFont is similar to UIFont in that you can create it with a name a size: ( plus a transform which we are passing NULL to )

```swift
CTFontRef myFont = CTFontCreateWithName( (CFStringRef)@"Helvetica", 12.0f, NULL);
    
CFRelease(myFont);
```

This will give us a 12pt normal Helvetica font, but what about bold?

```swift
CTFontRef myFont = CTFontCreateWithName( (CFStringRef)@"Helvetica-Bold", 12.0f, NULL);
    
CFRelease(myFont);
```

Here comes a similar problem to what we had with UIFont in that creating with a name requires the "real" or postscript font name. To do the above with only the family name of "Helvetica" we need to describe the font we would like in attributes using a font descriptor:

```swift
- (CTFontRef)createCTFontWithAttributes:(NSDictionary *)attributes 
{
    CTFontDescriptorRef descriptor = CTFontDescriptorCreateWithAttributes((CFDictionaryRef)attributes);
    CTFontRef font = CTFontCreateWithFontDescriptor(descriptor, 0, NULL);
    CFRelease(descriptor);
    return font;
}
  
-(CTFontRef)helveticaBoldFont
{
    NSDictionary *fontAttributes = @{
                                        (NSString *)kCTFontFamilyNameAttribute : @"Helvetica",
                                        (NSString *)kCTFontStyleNameAttribute : @"Bold",
                                        (NSString *)kCTFontSizeAttribute : [NSNumber numberWithFloat:12.0f]
                                    };
    
    return [self createCTFontWithAttributes:fontAttributes];
}
```

The above methods create an NSDictionary of font attributes and we use the CTFontDescriptorCreateWithAttributes to create a descriptor and then generate a CTFontRef from the the descriptor using: CTFontCreateWithFontDescriptor.

That seems like a lot of work just to get a bold font, however if we don't know the postscript font name for the font we want and/or need to generate it dynamically this can be very useful.

Now my very next thought was great, now lets do italic. But italic isn't an available font attribute it is a font trait. So now our simple create font with descriptor isn't going to work.

Apple walks through creating a font from a family name given traits in [listing 2-6](http://developer.apple.com/library/ios/#documentation/StringsTextFonts/Conceptual/CoreText_Programming/Operations/Operations.html#//apple_ref/doc/uid/TP40005533-CH4-SW4) of common operations in the documentation.

We could use this example as a basis for creating our own custom font descriptors but luckily someone has done all the hard work for us. Enter [DTCoreTextFontDescriptor](https://github.com/Cocoanetics/DTCoreText/blob/master/Core/Source/DTCoreTextFontDescriptor.h) !

DTCoreTextFontDescriptor is part of a greater [DTCoreText](https://github.com/Cocoanetics/DTCoreText) project kindly posted to github for all to use by Cocoanetics . The library is very good and I hightly recommend checking it out an reading through it to get some good ideas or even using it in full. However for my needs I have found using just the DTCoreTextFontDescriptor to be incredibly valuable.

Producing an italic helvetica font is a simple as:

```swift
DTCoreTextFontDescriptor *fontDescriptor = [[DTCoreTextFontDescriptor alloc] init];
 
fontDescriptor.fontFamily = @"Helvetica";
fontDescriptor.italicTrait = YES;
 
CTFontRef italicHelvetica = [fontDescriptor newMatchingFont];
 
CFRelease(italicHelvetica);
 
[fontDescriptor release];
```

Likewise creating the bold font is as easy as:

```swift
DTCoreTextFontDescriptor *fontDescriptor = [[DTCoreTextFontDescriptor alloc] init];
 
fontDescriptor.fontFamily = @"Helvetica";
fontDescriptor.boldTrait = YES;
 
CTFontRef boldHelvetica = [fontDescriptor newMatchingFont];
 
CFRelease(boldHelvetica);
    
[fontDescriptor release];
```

You want bold and italic, you may have already guessed:

```swift
DTCoreTextFontDescriptor *fontDescriptor = [[DTCoreTextFontDescriptor alloc] init];
 
fontDescriptor.fontFamily = @"Helvetica";
fontDescriptor.boldTrait = YES;
fontDescriptor.italicTrait = YES;
    
CTFontRef boldAndItalicHelvetica = [fontDescriptor newMatchingFont];
 
CFRelease(boldAndItalicHelvetica);
    
[fontDescriptor release];
```

With all the complexities of creating the proper font attributes and traits abstracted away we can get the CTFontRef we need and add it to our attributed strings and present nice rich text!


